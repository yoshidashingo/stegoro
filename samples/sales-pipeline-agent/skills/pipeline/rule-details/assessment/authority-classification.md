# A4: Authority Classification — Phase Rule

## Purpose

A3の優先度付きアクションリストの各アクションを、自動実行リストと承認制リストに分類する。`action-authority.md` の低リスク/高リスクテーブルを参照基準とし、境界曖昧ケースは保守的に承認制として扱う（Pitfall 2対策）。同一担当者への承認リクエストは最大3件ずつバッチ化し（Pitfall 3対策）、分類理由を監査ログに記録する。

**Classification**: ALWAYS EXECUTE（日中モード・対応要案件あり時に必須）
**Approval Gate**: あり（CP-3: 自動実行/承認制の境界が正しく適用されているか確認）
**参照元**: `workflow-architecture.md` A4 セクション

---

## Prerequisites

- A3の優先度付きアクションリスト（`state/action-list-prioritized.yaml`）が確定していること
- `config/action-authority.md` が参照可能であること
- S1のAPIモード（通常 / セーフ / 読み取り専用）が確定していること（読み取り専用時は自動実行を制限）

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: A3のアクションリストに1件以上のアクションが存在する場合
- Skip IF: アクション件数0件（このステージには到達しない）

---

## Execution Steps

### Step 1: 低リスク/高リスクテーブルの参照

`config/action-authority.md` に定義された権限テーブルを参照して各アクションを分類する。

**低リスク（自動実行可）アクション一覧**:

| アクション種別 | 具体的な操作 | 条件 |
|-------------|-----------|------|
| `record_activity` | Salesforce Activity（Task/Event）の新規記録 | 削除・上書きは不可 |
| `send_reminder` | 担当者へのSlack DM / 内部リマインダー | 社内宛のみ。顧客宛は高リスク |
| `generate_meeting_prep` | 会議準備サマリー生成・担当者送付 | 生成・送付先が担当者（社内）のみ |
| `update_stage_forward` | Salesforceステージの**前進**更新のみ | Prospecting→Qualification など |
| `send_deadline_alert` | 期限接近アラート（担当者宛） | 担当者Slack DM。顧客宛は高リスク |

**高リスク（承認制必須）アクション一覧**:

| アクション種別 | 具体的な操作 | リスク理由 |
|-------------|-----------|-----------|
| `send_email` | 顧客へのメール送信 | 顧客との関係に直接影響 |
| `update_amount` | 商談金額の変更 | 受注予測・売上計上に影響 |
| `update_stage_backward` | ステージの**後退**更新 | Negotiation→Proposal など |
| `close_deal` | 商談のClosed Won / Closed Lost | 取消不能な最終判定 |
| `convert_lead` | リードの商談化（Opportunity変換） | 新規商談の作成を伴う |
| `register_lead` | 新規リード登録 | 新規レコード作成 |
| `delete_record` | 既存レコードの削除 | 取消困難な破壊的操作 |
| `human_review` | 矛盾フラグ付き案件の確認依頼 | 人間判断が必要 |

### Step 2: 各アクションへの権限分類の適用

A3のアクションリストの各エントリに対して権限分類を適用する。

**分類ロジック**:
```
FOR each action IN action_list_prioritized:

  IF action.action_type IN low_risk_table:
    # ステージ更新は方向性を確認
    IF action.action_type == "update_stage":
      IF direction == "forward":
        authority = "auto"
      ELSE:  # backward または unknown
        authority = "approval_required"
        classification_reason = "ステージ後退は高リスク（Pitfall 2対策）"
    ELSE:
      authority = "auto"
      classification_reason = "低リスクアクション（action-authority.md 参照）"

  ELSE IF action.action_type IN high_risk_table:
    authority = "approval_required"
    classification_reason = "高リスクアクション: {action.action_type}（action-authority.md 参照）"

  ELSE:
    # 境界曖昧ケース: 保守的に承認制として扱う
    authority = "approval_required"
    classification_reason = "境界不明確のため保守的判定: 承認制（Pitfall 2対策）"
    ambiguous_flag = true
```

**読み取り専用モード時の追加制約**:
- `record_activity`（Salesforce書き込み）を含む全ての Salesforce 書き込みアクション: 自動実行から除外し繰り延べ
- `send_reminder`（Slack送信）: 読み取り専用モードの影響を受けない（実行可能）

### Step 3: 境界曖昧ケースの処理（Pitfall 2対策）

どちらか判断できないアクションは必ず承認制として扱い、曖昧フラグを記録する。

**よく発生する境界曖昧ケース**:

| ケース | 判定 | 理由 |
|--------|------|------|
| ステージ更新（方向性不明） | 承認制 | 後退リスクを排除できないため |
| 金額フィールドへのタッチ（コメント追加） | 承認制 | 金額変更と混同リスクあり |
| カスタムオブジェクトへの書き込み | 承認制 | 影響範囲が `action-authority.md` で未定義 |
| 複数オブジェクトにまたがる更新 | 承認制 | 波及リスクの評価が困難 |

**曖昧フラグの活用**:
- 曖昧フラグ付きアクションが蓄積した場合（月次で5件以上）、`action-authority.md` への追記候補として管理者に通知する

### Step 4: 承認リクエストのバッチ化（Pitfall 3対策）

同一担当者への承認制アクションを最大3件ずつバッチにまとめる。

**バッチ化ルール**:
```
FOR each owner_id IN unique_owners(approval_required_actions):
  owner_actions = filter(approval_required_actions, owner_id == owner_id)
  batches = chunk(owner_actions, size=3)  # 3件ずつ分割

  FOR each batch IN batches:
    create_approval_batch(
      batch_id = "BATCH-{owner_id}-{sequence}",
      actions = batch,
      recipient = owner_id
    )
```

**バッチ化の優先度順序**:
- 同一バッチ内ではP1 → P2 → P3 の順に並べる
- P1アクションが4件以上ある場合: P1のみのバッチを最初に送信し、残りをP2以降と混在させる

**バッチ例**:
```yaml
approval_batch:
  batch_id: "BATCH-U123456-001"
  recipient_slack_id: "U123456789"
  recipient_name: "田中 太郎"
  actions:
    - action_id: "ACT-20260405-003"
      priority: P1
      summary: "株式会社H — フォローアップメール（クロージング明日）"
    - action_id: "ACT-20260405-007"
      priority: P2
      summary: "株式会社F — ステージ更新提案（Proposal→Negotiation）"
    - action_id: "ACT-20260405-011"
      priority: P2
      summary: "株式会社I — リード登録提案"
```

### Step 5: 分類結果の確定とCP-3チェック

分類完了後、CP-3チェックを実施する。

**CP-3チェック項目**:
- [ ] 顧客向けメール送信アクションが全て承認制になっているか
- [ ] ステージ後退アクションが全て承認制になっているか
- [ ] 境界曖昧ケースが承認制として扱われているか
- [ ] 同一担当者への承認リクエストが3件以内のバッチにまとめられているか
- [ ] 分類理由が全アクションに記録されているか

**分類結果サマリーをサイクル状態ファイルに追記**:
```markdown
## Action Classification
- Auto-Execute: [X]件
- Approval Required: [X]件（バッチ数: [Y]件）
- Deferred（読み取り専用モード）: [X]件
- Ambiguous（保守的判定）: [X]件
```

---

## Output Artifacts

A4 が生成するアーティファクト:

1. **自動実行リスト**（`state/auto-execute-list.yaml`） — D1で参照
2. **承認制バッチリスト**（`state/approval-batch-list.yaml`） — D2で参照
3. **分類理由ログ** — 監査ログに追記
4. **曖昧フラグリスト** — 月次レビュー用に `config/action-authority.md` 更新候補として記録

---

## Completion Message

A4 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — A4: Authority Classification 完了

分類結果:
- 自動実行: [X]件（Activity記録: [X]件、リマインダー: [X]件、会議準備: [X]件）
- 承認制: [X]件（[Y]バッチ、担当者[Z]名宛）
- 保守的判定（境界曖昧）: [X]件 → 承認制として扱い
- 読み取り専用モードで繰り延べ: [X]件

CP-3チェック: [PASS / 要確認]

次のステップ:
- 低リスクあり → D1（Auto-Execute）へ続行
- 高リスクあり → D2（Approval Request）へ続行
- アクションなし → D4（Loop Control）へ

### WHAT'S NEXT
**A) 修正依頼** — 権限分類の基準や境界定義を修正する
**B) [D1 Auto-Execute / D2 Approval Request / D4 Loop Control]へ続行** — 分類結果に基づいて続行する
```

---

## Error Handling

### エラー 1: ステージ更新の方向性未確認による高リスクアクションの自動実行（Pitfall 2対策）

**症状**: 「ステージ更新」というアクション種別だけを見て前進/後退を区別せず一律に自動実行とし、Negotiation→Proposalの後退更新が自動実行される。担当者が気づかないまま商談ステージが後退し、受注予測が狂う。
**原因**: Step 2のステージ更新方向性チェックが実装されていない。
**対処**: `update_stage` アクションは必ず方向性（forward / backward）を確認し、backwardは承認制として分類する。

**GOOD例**:
```
アクション: update_stage, 対象: Proposal → Negotiation（前進）
→ 方向性チェック: forward
→ authority: auto（低リスク）
→ D1で自動実行

アクション: update_stage, 対象: Negotiation → Proposal（後退）
→ 方向性チェック: backward
→ authority: approval_required（高リスク）
→ D2で承認リクエスト送信（Pitfall 2対策）
```

**BAD例**:
```
アクション: update_stage（方向性未確認）
→ action_type = "update_stage" → 低リスクテーブルに一致
→ authority: auto
→ D1でNegotiation→Proposalの後退を自動実行
→ 担当者が気づかず受注確率が75%→50%に低下
→ 月次フォーキャストが狂い経営層への報告に支障（Pitfall 2対策漏れ）
```

### エラー 2: 境界曖昧ケースの自動実行（Pitfall 2対策）

**症状**: `action-authority.md` に未定義のカスタムオブジェクト更新アクションを「低リスク」と判断して自動実行し、関連する商談データが意図せず変更される。
**原因**: Step 3の境界曖昧ケース処理が実装されておらず、未定義アクションをデフォルトで自動実行に分類している。
**対処**: `action-authority.md` に定義されていないアクションタイプは全て承認制として扱う（保守的デフォルト）。

**GOOD例**:
```
アクション: update_custom_object_X（action-authority.md に未定義）
→ 低リスクテーブル: 未定義
→ 高リスクテーブル: 未定義
→ 境界曖昧ケース判定
→ authority: approval_required（保守的判定）
→ ambiguous_flag: true を記録
→ 月次レビュー候補として action-authority.md 更新を促進（Pitfall 2対策）
```

**BAD例**:
```
アクション: update_custom_object_X（action-authority.md に未定義）
→ 未定義のためデフォルト: auto と判定
→ D1でカスタムオブジェクトを自動更新
→ 関連する商談の集計値が変化、担当者が気づかない
→ 週次パイプラインレビューで不整合が発覚（Pitfall 2対策漏れ）
```

### エラー 3: 承認リクエストの分散送信による通知疲れ（Pitfall 3対策）

**症状**: 同一担当者へ10件の承認リクエストを1件ずつSlack DM送信し、担当者のDMが承認リクエストで埋め尽くされる。担当者が全件を却下または無視するようになる。
**原因**: Step 4のバッチ化処理が実装されていない。
**対処**: 同一担当者への承認リクエストは最大3件を1メッセージにまとめてから D2 に渡す。

**GOOD例**:
```
担当者 田中太郎宛 承認リクエスト: 7件
→ バッチ化: バッチ1（P1×1, P2×2）+ バッチ2（P2×1, P3×2）+ バッチ3（P3×1）
→ D2で3回のSlackメッセージを送信（7件個別ではなく3バッチ）
→ 担当者が受け取るメッセージ数: 7件 → 3件に削減（Pitfall 3対策）
```

**BAD例**:
```
担当者 田中太郎宛 承認リクエスト: 7件
→ バッチ化なし: 7件を個別にSlack DM送信
→ 担当者のDMが承認リクエストで埋まる
→ 「承認リクエストが多すぎて対応できない」と全件却下
→ 本当に重要なP1アクション（クロージング翌日）も見落とし（Pitfall 3対策漏れ）
```

### エラー 4: 読み取り専用モード中のSalesforce書き込み実行（Pitfall 4対策）

**症状**: S1で読み取り専用モードと判定されているにもかかわらず、A4での分類時にAPIモードを確認せず `record_activity`（Salesforce書き込み）を自動実行リストに含め、D1でDMLエラーが発生する。
**原因**: Step 2の読み取り専用モード追加制約チェックが実装されていない。
**対処**: A4の分類時にS1のAPIモードを確認し、読み取り専用モード時はSalesforce書き込み系アクションを繰り延べリストに移動する。

**GOOD例**:
```
S1 APIモード: 読み取り専用（Salesforce使用率 96%）
→ A4分類: record_activity → 自動実行候補
→ 読み取り専用チェック: 書き込みアクション → 繰り延べに移動
→ 繰り延べ理由: "read_only_mode_sf_write_blocked"
→ send_reminder（Slack）は読み取り専用の影響なし → 自動実行リストに維持
→ DMLエラーなし（Pitfall 4対策）
```

**BAD例**:
```
S1 APIモード: 読み取り専用（確認なし）
→ record_activity を自動実行リストに含めたままD1へ渡す
→ D1でSalesforce API呼び出し → DMLエラー発生
→ 個別アクション失敗でリトライが発生 → さらにAPI使用量を消費
→ ガバナーリミット枯渇リスクが増大（Pitfall 4対策漏れ）
```
