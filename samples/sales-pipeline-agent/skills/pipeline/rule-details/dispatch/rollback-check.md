# D5: Rollback Check — Phase Rule

## Purpose

本サイクルで実行した自動実行・承認済み実行アクションの結果を検証し、意図しない変更を検出した場合にロールバックを実行する。実行前スナップショットと実行後のSalesforceレコード状態を比較し、異常を検出した場合はSalesforce復元APIでロールバック、担当者へのSlack通知、監査ログへの記録を行う。異常なし・ロールバック不要の場合も検証完了記録は必須。

**Classification**: ALWAYS EXECUTE（DISPATCH フェーズ終了時に必須）
**Approval Gate**: あり（CP-5: 異常なし または ロールバック完了の確認）
**参照元**: `workflow-architecture.md` D5 セクション

---

## Prerequisites

- D1・D3の実行完了記録（監査ログ）が参照可能であること
- 実行前スナップショット（`state/pre-execution-snapshots/`）が存在すること
- Salesforceへの読み取りアクセスが可能であること（読み取り専用モードでも実行可能）
- ロールバックAPI（Salesforce Undelete / SOQL UPDATE）の呼び出し権限があること

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: 常に実行（D4の終了判定後）
- Skip IF: なし（スキップ不可。Salesforce書き込みが0件の場合も検証完了記録を残す）

---

## Execution Steps

### Step 1: 検証対象アクションの特定

本サイクルで実行した全Salesforce書き込みアクションをリストアップする。

**検証対象**:
- D1で実行した自動実行アクション（`record_activity`, `update_stage_forward`）
- D3で実行した承認済みアクション（`update_stage_backward`, `close_deal`, `convert_lead`, `register_lead`）

**検証不要**（Salesforce書き込みを伴わないもの）:
- `send_reminder`（Slack DM）
- `send_email`（Gmail）
- `generate_meeting_prep`（Slack送付）

**書き込みアクションが0件の場合**:
- 検証処理をスキップし、Step 5の「検証完了記録のみ」を実施してR1へ進む

### Step 2: 実行前スナップショットと実行後状態の比較

各検証対象アクションについて、実行前スナップショットと現在のSalesforceレコード状態を比較する。

**Salesforce現状取得**:
```
GET /services/data/v58.0/sobjects/Opportunity/{opportunity_id}
   ?fields=Id,StageName,Amount,CloseDate,LastActivityDate,IsDeleted
```

**比較する項目と異常検出パターン**:

| 比較項目 | 意図した変更 | 異常パターン |
|---------|-----------|------------|
| StageName | 前進更新（A→B）のみ想定 | B→A（後退）が発生していた |
| StageName | 後退更新（承認済み）で後退想定 | 意図と異なるステージ値 |
| Amount | 変更なし（record_activityのみ） | 金額が変化している |
| IsDeleted | false のまま想定 | true（意図しない削除） |
| Activities count | N+1（1件追加）想定 | カウントが増えていない（記録失敗） |

**比較ロジック**:
```
FOR each executed_action IN sf_write_actions:
  snapshot = load_snapshot(executed_action.action_id)
  current_state = salesforce.get(executed_action.target_opportunity_id)

  IF executed_action.action_type == "update_stage_forward":
    IF current_state.StageName != expected_new_stage:
      mark_anomaly(executed_action, "stage_mismatch",
                   expected=expected_new_stage, actual=current_state.StageName)

  IF executed_action.action_type == "record_activity":
    IF current_state.Amount != snapshot.pre_state.Amount:
      mark_anomaly(executed_action, "amount_unexpected_change",
                   expected=snapshot.pre_state.Amount, actual=current_state.Amount)

  IF current_state.IsDeleted == true AND snapshot.pre_state.IsDeleted == false:
    mark_anomaly(executed_action, "unexpected_deletion")
```

### Step 3: 異常検出時のロールバック実行

異常が検出された場合、アクション種別に応じたロールバックを実行する。

**ロールバック手順**:

| 異常パターン | ロールバック方法 |
|------------|----------------|
| ステージ意図せず後退 | SalesforceステージをスナップショットのStageName値に戻す（SOQL UPDATE） |
| 金額が意図せず変化 | Salesforce AmountをスナップショットのAmount値に戻す（SOQL UPDATE） |
| 意図しない削除 | Salesforce Undelete API で復元 |
| Activity記録の重複 | 重複したTask/EventレコードをSalesforceから削除（SOQL DELETE） |

**ロールバックAPIの呼び出し**:

ステージ・金額の修正:
```
PATCH /services/data/v58.0/sobjects/Opportunity/{opportunity_id}
{
  "StageName": "{snapshot.pre_state.StageName}",
  "Amount": {snapshot.pre_state.Amount}
}
```

削除レコードの復元:
```
POST /services/data/v58.0/undelete
{
  "ids": ["{opportunity_id}"]
}
```

**ロールバック成功時の後処理**:
1. 担当者にSlack DM通知を送信
2. 監査ログに記録
3. 当該アクション種別の自動実行を **一時停止** フラグを立てる（再発防止）

**担当者Slack通知フォーマット**:
```
⚠️【自動修正通知】エージェントによる実行を取り消しました

対象商談: {opportunity_name}
取り消した操作: {action_type}（{詳細}）
理由: 意図しない変更を検出しました
　実行前: {snapshot.pre_state の概要}
　実行後（検出値）: {current_state の概要}

Salesforceを元の状態に戻しました。
ご不明な点は #sales-pipeline-ops までご連絡ください。
```

**一時停止フラグ**:
```yaml
# state/auto-execute-suspend.yaml
suspended_actions:
  - action_type: "update_stage_forward"
    suspended_at: "2026-04-05T10:50:00+09:00"
    reason: "rollback_triggered"
    target_opportunity_id: "006xxxxxxxxxxxx"
    resume_at: null  # 管理者による手動解除まで停止
```

### Step 4: ロールバック失敗時の緊急処置

ロールバックAPIの呼び出しが失敗した場合の緊急処置を実施する。

**ロールバック失敗時の対応**:
1. 管理者緊急通知（`#sales-pipeline-ops` Slack + メール）を即時送信
2. 手動修正ガイドを通知に含める
3. 当該商談に関連する全アクションの自動実行を一時停止
4. 監査ログに「ロールバック失敗」を記録

**管理者緊急通知フォーマット**:
```
🚨【緊急】ロールバック失敗 — 手動対応が必要です

商談: {opportunity_name}（ID: {opportunity_id}）
問題: {異常の説明}
ロールバック失敗理由: {APIエラーメッセージ}

手動修正手順:
1. Salesforceにログインし、商談「{opportunity_name}」を開く
2. {修正すべきフィールドと値} を手動で修正してください
   修正前の値: {snapshot.pre_state の詳細}
3. 修正完了後、#sales-pipeline-ops に完了報告をお願いします

エージェントは当該商談の自動実行を停止しました。
```

### Step 5: 検証完了記録

ロールバックの有無にかかわらず、検証完了記録を監査ログに残す。

**正常時（異常なし）**:
```yaml
rollback_check_result:
  cycle_id: "cycle-uuid-xxxx"
  checked_at: "2026-04-05T10:50:00+09:00"
  checked_actions_count: 5
  anomalies_detected: 0
  rollbacks_executed: 0
  status: "verified_ok"
```

**異常あり（ロールバック実行）**:
```yaml
rollback_check_result:
  cycle_id: "cycle-uuid-xxxx"
  checked_at: "2026-04-05T10:50:00+09:00"
  checked_actions_count: 5
  anomalies_detected: 1
  rollbacks_executed: 1
  rollback_details:
    - action_id: "ACT-20260405-003"
      anomaly_type: "stage_mismatch"
      expected: "Negotiation"
      actual: "Proposal"
      rollback_status: "success"
      rollback_completed_at: "2026-04-05T10:51:00+09:00"
  status: "rollback_completed"
```

**CP-5チェック**:
- [ ] 全実行アクションの検証が完了しているか
- [ ] 異常検出時にロールバックまたは管理者通知が実施されているか
- [ ] 検証完了記録が監査ログに追記されているか

---

## Output Artifacts

D5 が生成するアーティファクト:

1. **ロールバック検証結果**（監査ログ追記） — R3で参照
2. **一時停止フラグ**（`state/auto-execute-suspend.yaml`） — 次サイクルのD1・A4で参照
3. **異常検出通知** — 担当者・管理者へのSlack通知（異常あり時のみ）

---

## Completion Message

D5 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — D5: Rollback Check 完了

検証結果:
- 検証対象: [X]件
- 異常検出: [X]件
- ロールバック実行: [X]件（成功: [X]件 / 失敗: [X]件）
- 一時停止フラグ設定: [X]件のアクション種別

CP-5チェック: [PASS / 異常あり（詳細確認要）]

### WHAT'S NEXT
**A) 修正依頼** — ロールバック対象の判定基準や復元方法を修正する
**B) [R1 Daily Summary Generation]へ続行** — 日次サマリー生成へ進む
```

---

## Error Handling

### エラー 1: 実行結果未検証による誤ったステータス更新の放置（Pitfall 2対策）

**症状**: 自動実行完了後に実行結果を検証せず、誤ったステージ更新がそのまま残存した。翌日の巡回で発覚したが、その間に顧客との商談進行に支障が出た。
**原因**: D5をスキップした、または Step 2の比較ロジックが実装されていない。
**対処**: D5は ALWAYS EXECUTE（スキップ不可）として設計し、書き込みアクションが0件の場合も検証完了記録を残す。

**GOOD例**:
```
D1: update_stage_forward（Proposal → Negotiation）実行
→ D5: スナップショット（StageName: Proposal）と現状（StageName: Qualification）を比較
→ 想定（Negotiation）と異なる（Qualification）→ 異常検出
→ ロールバック: StageName を Proposal に戻す
→ 担当者Slack通知: 「ステージ更新を取り消しました」
→ 翌日の商談進行に影響なし（Pitfall 2対策）
```

**BAD例**:
```
D1: update_stage_forward 実行
→ D5をスキップ（実行後検証なし）
→ 誤ったステージQualificationのまま放置
→ 翌日の巡回でQualificationステージでの停滞として誤検出
→ 担当者に誤った停滞アラート送信 → 混乱（Pitfall 2対策漏れ）
```

### エラー 2: ロールバック失敗時の無通知放置

**症状**: SalesforceロールバックAPIが失敗したにもかかわらず管理者への通知を送らず、誤ったデータが修正されないまま次サイクルに進む。
**原因**: Step 4の緊急処置が実装されていない。
**対処**: ロールバック失敗は「手動対応が必要な緊急事態」として、Slack通知とメールの両方で管理者に即時報告する。エラーを隠蔽しない。

**GOOD例**:
```
ロールバックAPI呼び出し → 403 Forbidden（権限不足）
→ 管理者緊急通知（Slack #sales-pipeline-ops + メール）を即時送信
→ 手動修正手順を通知に含める
→ 当該商談の自動実行を一時停止
→ 管理者が翌朝に手動修正を実施
→ 修正後に一時停止を解除
```

**BAD例**:
```
ロールバックAPI → 403 Forbidden
→ エラーをログに記録するのみ（通知なし）
→ 誤ったデータで翌サイクル以降の検出・アクションが実行される
→ 問題が連鎖的に拡大し、複数の商談データが不整合状態に
```

### エラー 3: 一時停止フラグなしによる同じ異常の繰り返し（Pitfall 2対策）

**症状**: update_stage_forward でロールバックが発生したにもかかわらず一時停止フラグを設定せず、次サイクルで同じアクションが再実行され再びロールバックが発生するループに陥る。
**原因**: Step 3のロールバック後処理で一時停止フラグ設定が省略されている。
**対処**: ロールバック発生時は当該アクション種別・商談IDの組み合わせに一時停止フラグを設定し、管理者が手動解除するまで再実行を防ぐ。

**GOOD例**:
```
update_stage_forward（商談006xxx）でロールバック発生
→ auto-execute-suspend.yaml に記録:
   action_type: "update_stage_forward", opportunity_id: "006xxx"
→ 次サイクルのA4: この商談・種別は一時停止中 → 承認制に変更（保守的判定）
→ 担当者に確認を取ってから再実行
→ 同じ異常の繰り返しなし（Pitfall 2対策）
```

**BAD例**:
```
update_stage_forward でロールバック発生（一時停止フラグなし）
→ 次サイクル: 同じ商談で同じアクションを自動実行
→ 再びロールバック発生
→ サイクルごとに「更新 → ロールバック」を繰り返す無限ループ
→ 商談データが毎時間変動し、担当者が状態を把握できなくなる（Pitfall 2対策漏れ）
```

### エラー 4: スナップショット不在時の検証スキップ

**症状**: D1・D3の実行前スナップショットが何らかの理由で保存されていなかったため、ロールバック判定ができず検証をスキップする。実際に異常が発生していたが見逃す。
**原因**: `state/pre-execution-snapshots/{action_id}.yaml` が存在しない場合の代替処理が未実装。
**対処**: スナップショットが存在しない場合は「現在の状態のみのスナップショット」を事後作成し、少なくともIsDeletedとStageNameの合理性チェック（既知の有効なステージ値かどうか）を実施する。

**GOOD例**:
```
ACT-20260405-003 のスナップショット不在
→ 現時点のSalesforceレコードを取得
→ 合理性チェック:
   - StageName が有効なステージ値（config/valid-stages.yaml）に含まれるか
   - IsDeleted = false か
   - Amount が正の値か
→ 全チェック通過 → 「スナップショット不在のため限定的検証実施」として記録
→ 管理者に「スナップショット取得失敗」のログを通知（D1の改善を促す）
```

**BAD例**:
```
ACT-20260405-003 のスナップショット不在
→ 検証処理をスキップ
→ 「検証完了（スナップショットなし）」とだけ記録
→ 実際にIsDeletedがtrueになっていた（意図しない削除）を見逃す
→ 次サイクルでその商談のデータを参照しようとしてNullReferenceError
```
