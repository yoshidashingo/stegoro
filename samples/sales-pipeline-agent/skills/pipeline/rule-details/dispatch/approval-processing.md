# D3: Approval Processing — Phase Rule

## Purpose

D2で送信した承認リクエストへの返答（承認 / 却下）を取得し、承認済みアクションを実行する。夜間モード保留フラグのチェック、D1と同様の排他制御・スナップショット取得、却下理由の記録と自動判定ロジック見直しフラグの管理を行う。承認処理の完全な監査証跡を残す。

**Classification**: CONDITIONAL（承認済みアクションがある場合）
**Approval Gate**: あり（CP-4: 承認内容と実行予定の一致確認）
**参照元**: `workflow-architecture.md` D3 セクション

---

## Prerequisites

- D2の承認リクエスト送信記録（`state/approval-requests.yaml`）が確定していること
- Slackインターアクティビティ（Webhook）またはメール返信の結果が取得可能であること
- D2の夜間保留フラグ情報が参照可能であること
- D1と同様の排他制御ロック機構が利用可能であること

---

## Execution Classification

**CONDITIONAL EXECUTE**

- Execute IF: 承認リクエストへの返答（承認 / 却下）が1件以上存在する場合
- Skip IF: 承認待ちアクションが全て未応答の場合 → D4（Loop Control）へ直行

---

## Execution Steps

### Step 1: 承認結果の取得

Slack インターアクティビティWebhookまたはメール返信から承認 / 却下の結果を取得する。

**Slackインターアクティビティから取得**:
```json
{
  "type": "block_actions",
  "actions": [{
    "action_id": "approve_all",
    "value": "APPR-20260405-001"
  }],
  "user": {
    "id": "U123456789",
    "name": "田中太郎"
  }
}
```

**取得できる結果パターン**:

| アクション | 意味 | 処理 |
|----------|------|------|
| `approve_all` | バッチ内全件承認 | バッチ内全アクションを承認済みに |
| `reject_all` | バッチ内全件却下 | バッチ内全アクションを却下に。却下理由の入力を促す |
| `individual_review` → `approve_{action_id}` | 個別承認 | 指定アクションのみ承認済みに |
| `individual_review` → `reject_{action_id}` | 個別却下 | 指定アクションのみ却下。却下理由を記録 |

**結果を `state/approval-requests.yaml` に更新**:
```yaml
- request_id: "APPR-20260405-001"
  status: "partially_approved"
  responded_by: "U123456789"
  responded_at: "2026-04-05T10:25:00+09:00"
  action_results:
    - action_id: "ACT-20260405-003"
      result: "approved"
    - action_id: "ACT-20260405-007"
      result: "rejected"
      rejection_reason: "顧客との交渉で既にステージ変更を保留しています"
```

### Step 2: CP-4チェック（承認内容と実行予定の一致確認）

承認された内容が実際に実行しようとしているアクションと一致していることを確認する。

**CP-4チェック項目**:
- [ ] 承認されたアクションIDが `state/action-list-prioritized.yaml` に存在するか
- [ ] 承認されたアクションの対象商談IDとSalesforceの現在の状態に大きな変化がないか
  - 例: 承認時にはProposalステージだったが、実行前にNegotiationに変更されていた場合
- [ ] 夜間保留フラグが付与されているアクションは実行時刻が `execute_after` を過ぎているか

**Salesforce現状確認**（承認から15分以上経過している場合）:
```
GET /services/data/v58.0/sobjects/Opportunity/{opportunity_id}
→ 現在のStageName、Amount を取得して承認時の状態と照合
```

変化が大きい場合（ステージが変わっていた等）は担当者に確認メッセージを送信して実行を一時保留する。

### Step 3: 承認済みアクションの実行

D1と同様の排他制御・スナップショット取得を適用した上で実行する。

**実行前の排他制御**（D1 Step 1と同様）:
- 同一 `opportunity_id` のロックを取得
- TTL 120秒のロックを設定

**実行前スナップショットの取得**（D1 Step 2と同様）:
- Salesforce書き込みを伴うアクションの実行前に現状をキャプチャ

**夜間保留チェック**:
```
IF action.deferred_execution.execute_after > current_time:
  → 実行を保留。送信キューに登録
  → 担当者通知: 「承認を受け付けました。送信は {execute_after} に実行します」
  CONTINUE  # 実行はスキップ
ELSE:
  → 通常実行
```

**承認済みアクション種別ごとの実行**:

| アクション種別 | 実行内容 | API |
|-------------|---------|-----|
| `send_email` | 下書きメールを顧客に送信 | Gmail API `messages.send` |
| `update_stage_backward` | Salesforceステージを後退更新 | Salesforce SOQL UPDATE |
| `close_deal` | Salesforce商談をClosed Won/Lostに更新 | Salesforce SOQL UPDATE |
| `convert_lead` | SalesforceリードをOpportunityに変換 | Salesforce `Lead.convertLead()` |
| `register_lead` | 新規リードをSalesforceに登録 | Salesforce SOQL INSERT |
| `human_review` | 担当者への確認通知送信 | Slack API |

**Salesforceセーフモード中の制約**:
- セーフモード（API使用率80-95%）: 承認済みアクションでもSalesforce書き込みを保留し次サイクルへ繰り延べ
- 通知系（send_email, Slack通知）はセーフモードの影響なし（Salesforce APIを消費しない）

### Step 4: 却下処理

却下されたアクションの処理を行う。

**却下時の処理フロー**:
```
1. 却下理由を state/approval-requests.yaml に記録
2. 同一アクション種別の却下累積カウントを更新
   rejection_count[action_type] += 1
3. 却下されたアクションは同サイクルで再提案しない（Pitfall 3対策）
4. 却下フラグを付与して action-list から除外
```

**自動判定ロジック見直しフラグ**:
```
IF rejection_count[action_type] >= 3 (同一パターンの却下が3回以上):
  → review_flag: true を state/rejection-patterns.yaml に記録
  → 管理者通知: 「{action_type} の却下率が高くなっています。
                   action-authority.md または判定ロジックの見直しを検討してください」
```

**却下理由の分類**（監査ログへの記録に使用）:

| 却下理由カテゴリ | 例 |
|---------------|---|
| `context_mismatch` | 交渉状況が既に変わっている |
| `timing_wrong` | 今は連絡を控えたい |
| `content_incorrect` | メール文面に誤りがある |
| `not_needed` | 既に別途対応済み |
| `other` | 自由記述 |

### Step 5: 実行完了記録

承認済み実行の完全な監査証跡を記録する。

```yaml
audit_log_entry:
  - action_id: "ACT-20260405-003"
    action_type: "send_email"
    target_opportunity_id: "006xxxxxxxxxxxx"
    executor: "human_approved"
    approved_by: "田中太郎（U123456789）"
    approved_at: "2026-04-05T10:25:00+09:00"
    executed_at: "2026-04-05T10:26:00+09:00"
    result: "completed"
    gmail_message_id: "msg_abc123"

  - action_id: "ACT-20260405-007"
    action_type: "update_stage_backward"
    executor: "rejected"
    rejected_by: "田中太郎（U123456789）"
    rejected_at: "2026-04-05T10:25:00+09:00"
    rejection_reason: "顧客との交渉で既にステージ変更を保留しています"
    rejection_category: "context_mismatch"
```

---

## Output Artifacts

D3 が生成するアーティファクト:

1. **承認済み実行完了記録**（監査ログ追記） — D5・R3で参照
2. **却下理由記録** — 自動判定ロジック見直しフラグに活用
3. **実行前スナップショット**（`state/pre-execution-snapshots/`） — D5のロールバック判定に使用
4. **夜間保留キュー** — 翌朝8:00実行用キュー

---

## Completion Message

D3 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — D3: Approval Processing 完了

処理結果:
- 承認済み実行完了: [X]件
- 夜間保留（翌朝8:00実行）: [X]件
- 却下: [X]件
  - 却下理由トップ: [理由カテゴリ]
  - 自動判定見直しフラグ: [なし / 発動（X件蓄積）]
- 未応答（D4で管理）: [X]件

CP-4チェック: [PASS / 要確認（変化検出 X件）]

### WHAT'S NEXT
**A) 修正依頼** — 承認処理の結果や判定ロジックを修正する
**B) [D4 Loop Control]へ続行** — 残件チェックとループ判定へ進む
```

---

## Error Handling

### エラー 1: 夜間承認後の顧客向けメール即時実行（ドメイン固有）

**症状**: 承認済みの顧客フォローアップメールを夜間22:00に即時実行した。顧客が深夜にメールを受信し不快感を示した。
**原因**: Step 3の夜間保留チェックが実装されていない。
**対処**: `send_email` アクションは実行前に `deferred_execution.execute_after` を確認し、時刻未到達の場合は送信キューに登録する。

**GOOD例**:
```
承認時刻: 22:05（夜間モード）
→ send_email アクション: execute_after = "2026-04-06T08:00:00+09:00"
→ D3: execute_after > 現在時刻 → 送信を保留
→ 担当者通知: 「承認を受け付けました。送信は 4/6 8:00 に実行します」
→ 翌朝8:00に自動送信
→ 顧客は適切な時間帯にメールを受信
```

**BAD例**:
```
承認時刻: 22:05（夜間モード）
→ 夜間保留チェックなし → 即時実行
→ 顧客が深夜22:07にフォローアップメールを受信
→ 顧客: 「深夜にメールを送ってくる会社と契約したくない」と返信
→ ¥12M商談の信頼関係が毀損（夜間制約違反）
```

### エラー 2: 却下されたアクションの同サイクル再提案（Pitfall 3対策）

**症状**: 担当者が「ステージ更新提案」を却下したにもかかわらず、次のループで同じ商談の同じアクションが再度承認リクエストとして送信される。
**原因**: Step 4の却下フラグ管理が未実装。却下済みアクションが action-list に残ったまま次ループで再評価される。
**対処**: 却下されたアクションには `status: "rejected_this_cycle"` フラグを付与し、D4のループ継続時に同サイクルでの再生成を禁止する。

**GOOD例**:
```
ACT-007「ステージ更新提案」が却下
→ status: "rejected_this_cycle" を付与
→ D4でループ継続 → A1への戻り
→ A2: ACT-007と同一パターンの検出が再度発生
→ rejected_this_cycle チェック: 同サイクルで却下済み → 生成スキップ
→ 担当者への重複承認リクエスト送信なし（Pitfall 3対策）
```

**BAD例**:
```
ACT-007「ステージ更新提案」が却下
→ 却下フラグ未付与
→ D4でループ継続 → A1への戻り
→ A2: 同じ商談・同じアクションを再生成
→ D2: 同じ担当者に再度承認リクエスト送信
→ 担当者: 「さっき却下したのにまた来た」と不信感増大（Pitfall 3対策漏れ）
```

### エラー 3: 承認後の状態変化による意図しない更新（ドメイン固有）

**症状**: 担当者がProposalステージからNegotiationへの更新を承認したが、承認から20分後の実行時点で別の担当者が既にSalesforceでNegotiationに更新済み。D3がNegotiationステージを重複して更新しようとする。
**原因**: Step 2のCP-4チェック（Salesforce現状確認）が実装されていない。
**対処**: 承認から15分以上経過したアクションは実行前にSalesforceの現状を確認し、状態が既に変更されている場合は実行をスキップする。

**GOOD例**:
```
承認: 10:25 / 実行: 10:48（23分後）
→ CP-4チェック: Salesforce現在のStageName = "Negotiation"
→ 承認時の状態: "Proposal" → 変化を検出
→ 実行をスキップ（既に変更済み）
→ 担当者通知: 「ステージ更新は既に実施済みのためスキップしました」
→ 重複更新なし
```

**BAD例**:
```
承認: 10:25 / 実行: 10:48（23分後）
→ Salesforce現状確認なし
→ StageName を "Proposal" → "Negotiation" に更新（既にNegotiationなのに）
→ Salesforce 記録: ステージが一時的に無意味な更新履歴を残す
→ D5のロールバックチェックで「想定外の更新」として検出され混乱
```

### エラー 4: 3回蓄積なしの見直しフラグ発動（誤検出）

**症状**: 却下が1回発生するたびに「自動判定ロジックの見直し」が管理者に通知される。1日に多数の却下通知が発生し、管理者が通知疲れになる。
**原因**: Step 4の却下累積カウントが1回でフラグを発動している。
**対処**: 同一アクション種別の却下が **3回以上蓄積** した場合のみ見直しフラグを発動する。カウントはサイクルをまたいで30日間累積する。

**GOOD例**:
```
send_email の却下: 1回目 → カウント: 1 / 3（フラグなし）
send_email の却下: 2回目 → カウント: 2 / 3（フラグなし）
send_email の却下: 3回目 → カウント: 3 / 3
→ 管理者通知: 「send_email の却下が3回蓄積。判定ロジックの見直しを検討してください」
→ 適切なタイミングでのみ通知（Pitfall 3対策）
```

**BAD例**:
```
send_email が1回却下
→ 即座に管理者通知: 「自動判定ロジックの見直しが必要です」
→ 1日に10件の却下 → 10件の管理者通知
→ 管理者が通知を無視するようになる（Pitfall 3対策漏れ）
```
