# S4: Change Detection — Phase Rule

## Purpose

S3で構築した統合ビューを解析し、7種の変更・異常を検出する。停滞商談・未返信メール・会議準備・金額変動・期限接近・未追跡リード・ステータス未反映の各ルールを適用し、検出結果リストを確定する。単一ソースの状態のみに依存した誤検出を防ぐため、多層判定（Pitfall 1対策）と重複通知抑制（Pitfall 3対策）を必ず適用する。

**Classification**: ALWAYS EXECUTE（日中モード時に必須）
**Approval Gate**: あり（CP-2: 検出件数・種別・多層判定結果の妥当性確認）
**参照元**: `workflow-architecture.md` S4 セクション

---

## Prerequisites

- S3の統合ビュー（`state/integrated-view.yaml`）が確定していること
- S3の矛盾フラグリストが参照可能であること
- 前回検出済みリスト（`state/last-detection-log.json`）が参照可能であること（重複通知抑制用）
- ステージ別滞留閾値（`config/stage-thresholds.yaml`）が設定済みであること

---

## Execution Classification

**ALWAYS EXECUTE**（日中モード時）

- Execute IF: S3の統合ビューに1件以上の商談データが存在する場合
- Skip IF: S3でデータ収集件数が0件（全ソースのデータが空）の場合 → R1へ直行

---

## Execution Steps

### Step 1: 矛盾フラグ付き案件の除外

S3で `conflict_flags` が付与されている案件を7種の検出ルール適用対象から除外する。

**除外処理**:
- `conflict_flags` が空でない商談は全検出ルールをスキップ
- 代わりに「人間確認依頼」として検出リストに追加（種別: `conflict_review`）
- 除外件数を検出サマリーに記録

```yaml
detection_results:
  - type: conflict_review
    opportunity_id: "006xxxxxxxxxxxx"
    opportunity_name: "株式会社A — Enterpriseプラン導入"
    conflict_types: ["conflict_stale_but_active"]
    priority: medium
    message: "Salesforceでは停滞判定対象ですが、Slackで直近にアクティビティが確認されています。担当者の確認が必要です。"
```

### Step 2: 検出ルール1 — 停滞商談（Pitfall 1対策: 多層判定必須）

**判定条件（AND条件 — 両方満たす場合のみ停滞として検出）**:

条件A: Salesforceのステージ滞留日数が閾値を超過している
```yaml
# config/stage-thresholds.yaml のデフォルト値
stage_stale_thresholds:
  Prospecting: 21
  Qualification: 14
  Proposal: 14
  Negotiation: 7
  Value Proposition: 14
  Id. Decision Makers: 14
  Perception Analysis: 14
  Needs Analysis: 14
  default: 21  # 上記以外のステージ
```

条件B: Gmail・Slack・Calendarで滞留閾値の半分の期間内にアクティビティが **ない**
```
アクティビティ確認期間 = ステージ別閾値 / 2
例: Proposalステージ 14日閾値 → 7日以内のGmail/Slack/Calendarアクティビティを確認
```

**多層判定のロジック**:
```
IF days_in_stage >= stage_threshold
  AND latest_activity_date < (today - stage_threshold / 2)
  AND conflict_flags = []
THEN: 停滞商談として検出
ELSE IF days_in_stage >= stage_threshold
  AND latest_activity_date >= (today - stage_threshold / 2)
THEN: conflict_stale_but_active フラグ（S3で付与済み）として除外済み
```

**検出エントリの生成**:
```yaml
- type: stale_deal
  opportunity_id: "006xxxxxxxxxxxx"
  opportunity_name: "株式会社B — ライセンス更新"
  stage: "Proposal"
  days_in_stage: 16
  threshold: 14
  last_activity_date: "2026-03-18"
  priority: high
  message: "Proposalステージに16日間停滞（閾値: 14日）。最終アクティビティ: 3/18のメール送信。Gmail/Slack/Calendarでも直近7日のアクティビティなし。"
```

### Step 3: 検出ルール2 — 未返信メール

**判定条件**:
- 担当者が送信したメール（Gmail `is:sent`）への返信（同一Thread-Idの受信メール）が3営業日以上存在しない
- 既に未返信リマインダーを送信済みの場合: 前回送信から4時間以内は重複通知抑制（Pitfall 3対策）

**3営業日の算出**:
- 土日・祝日を除いた営業日のみカウント
- 送信日当日はカウントしない（翌営業日から1日目）

**重複通知抑制チェック**:
```
IF last_detection_log にこの Thread-Id の未返信通知が存在
  AND (current_time - last_notification_sent) < 4時間
THEN: スキップ（重複通知として除外）
```

**検出エントリの生成**:
```yaml
- type: unanswered_email
  opportunity_id: "006xxxxxxxxxxxx"
  thread_id: "thread_def456"
  sent_date: "2026-04-01"
  business_days_without_reply: 3
  email_subject: "提案書ご確認のお願い"
  priority: medium
  message: "「提案書ご確認のお願い」への返信が3営業日（4/1送信）ありません。"
```

### Step 4: 検出ルール3 — 会議準備

**判定条件**:
- Google Calendarで外部参加者（自社ドメイン以外）を含む予定が現在時刻から24時間以内に存在する
- かつ、S3でその予定に `related_opportunity_id` が付与されている（商談との紐づき確認済み）

**会議準備サマリー生成の優先度**:
- 12時間以内の予定: P1（Critical）
- 12〜24時間以内の予定: P2（High）

```yaml
- type: meeting_prep
  opportunity_id: "006xxxxxxxxxxxx"
  event_id: "cal_xyz789"
  event_summary: "株式会社A — 提案レビュー"
  event_start: "2026-04-06T10:00:00+09:00"
  hours_until_meeting: 20
  priority: high
  message: "20時間後に「株式会社A — 提案レビュー」が予定されています。会議準備サマリーの生成が必要です。"
```

### Step 5: 検出ルール4 — 金額変動

**判定条件**:
- SalesforceのOpportunity `Amount` フィールドが前回巡回時の記録値から **10%以上** 変動している

**変動率算出**:
```
amount_change_rate = |current_amount - previous_amount| / previous_amount × 100
```

**閾値以下の変動（10%未満）は検出対象外**（軽微な変動による通知ノイズを防止）

```yaml
- type: amount_changed
  opportunity_id: "006xxxxxxxxxxxx"
  previous_amount: 5000000
  current_amount: 4000000
  change_rate: -20
  change_direction: "decreased"
  priority: high
  message: "商談金額が ¥5,000,000 から ¥4,000,000 に変動（-20%）しました。"
```

### Step 6: 検出ルール5 — 期限接近

**判定条件**:
- Opportunity `CloseDate` まで **7日以内**（本日を含む）

**優先度分類**:
| クロージング日まで | 優先度 | 通知方法 |
|----------------|--------|---------|
| 1日以内（当日〜翌日） | P1（Critical） | Slack DM即時通知 |
| 2〜7日以内 | P2（High） | 次回巡回サマリー集約 |

**重複通知抑制（Pitfall 3対策）**:
- 同一商談の期限接近通知は前回送信から **48時間** 以内は再送しない
- ただし P1（1日以内）は重複抑制を **適用しない**（緊急のため毎サイクル通知）

```yaml
- type: deadline_approaching
  opportunity_id: "006xxxxxxxxxxxx"
  opportunity_name: "株式会社C — Pro契約"
  close_date: "2026-04-07"
  days_to_close: 2
  amount: 3000000
  priority: high
  message: "クロージング日（4/7）まで2日です。商談「株式会社C — Pro契約」（¥3,000,000）の対応状況を確認してください。"
```

### Step 7: 検出ルール6 — 未追跡リード

**判定条件**:
- Gmail送受信または Google Calendar予定の参加者に、自社ドメイン以外のメールアドレスが含まれている
- かつ、そのメールドメインまたはメールアドレスがSalesforceのリード・商談・取引先に存在しない

**誤検出防止**:
- 既知の非見込み先ドメイン（`config/excluded-domains.yaml`: Gmail.com、hotmail.com 等のフリーメールアドレス）は除外
- 社内ツール・SaaSサービスの通知メール（no-reply@、noreply@、notifications@ 等）は除外

```yaml
- type: untracked_lead
  email: "suzuki@prospect-company.co.jp"
  domain: "prospect-company.co.jp"
  first_contact_date: "2026-04-03"
  contact_source: "gmail"
  email_subject: "貴社サービスについてのお問い合わせ"
  priority: medium
  message: "新規見込み先 prospect-company.co.jp（鈴木様）とのやり取りを検出。Salesforceにリードが存在しません。リード登録を提案します。"
```

### Step 8: 検出ルール7 — ステータス未反映

**判定条件**:
- Gmail送受信・CalendarミーティングがあるがSalesforce ActivityログにS3収集後24時間以上記録されていない

**ステータス未反映の証跡チェック**:
```
FOR each gmail_activity IN integrated_view[opportunity].gmail_activities:
  IF activity.date < (now - 24h)
    AND NOT EXISTS salesforce_activity
        WHERE date BETWEEN (activity.date - 2h) AND (activity.date + 2h)
        AND type IN ('Email', 'Call', 'Meeting')
  THEN: ステータス未反映として検出
```

**24時間バッファ**: Salesforceへの手動記録に猶予を持たせる（即時検出しない）

```yaml
- type: status_not_recorded
  opportunity_id: "006xxxxxxxxxxxx"
  activity_type: "email"
  activity_date: "2026-04-03T14:00:00+09:00"
  activity_source: "gmail"
  activity_description: "「提案書送付完了」メールを送信"
  priority: low
  message: "2026/4/3のGmailアクティビティ（提案書送付）がSalesforce Activityに記録されていません。自動記録を提案します。"
```

### Step 9: 1商談1アクション原則の適用（Pitfall 3対策）

同一商談に複数の検出結果がある場合、最高優先度の検出結果のみを残し、他はコンテキストとして付加する。

**優先度順序（高い順）**:
1. `conflict_review`（矛盾フラグ — 人間判断必須）
2. `deadline_approaching`（P1: 1日以内）
3. `stale_deal`（ヘルススコアRed）
4. `deadline_approaching`（P2: 2〜7日）
5. `amount_changed`
6. `unanswered_email`
7. `meeting_prep`
8. `untracked_lead`
9. `status_not_recorded`

**統合処理例**:
```yaml
# 統合前（同一商談に3件）
- type: unanswered_email, opportunity_id: "006xxx"
- type: deadline_approaching, opportunity_id: "006xxx"
- type: amount_changed, opportunity_id: "006xxx"

# 統合後（1件 + コンテキスト）
- type: deadline_approaching  # 最高優先度
  opportunity_id: "006xxx"
  additional_context:
    - "未返信メール: 提案書確認依頼（3営業日経過）"
    - "金額変動: ¥5M → ¥4M（-20%）"
  priority: high
```

### Step 10: 検出結果リストの確定とCP-2チェック

全検出ルールの適用後、検出結果リストを確定し、CP-2チェックを実施する。

**CP-2チェック項目**:
- [ ] 矛盾フラグ付き案件が全て `conflict_review` として除外されているか
- [ ] 多層判定が全ての停滞商談検出に適用されているか（単一ソース判定がないか）
- [ ] 重複通知抑制が適用されているか（4時間ルール・48時間ルール）
- [ ] 1商談1アクション原則が適用されているか

**CP-2合格基準**: 上記全て確認済み、かつ検出件数が0〜合理的な上限値（1サイクルあたり50件以内）であること

---

## Output Artifacts

S4 が生成するアーティファクト:

1. **検出結果リスト**（`state/detection-results.yaml`） — A1・A2・A3・A4で参照
2. **検出サマリー**（サイクル状態ファイルに追記） — ループ制御・R1サマリーで参照
3. **前回検出ログの更新**（`state/last-detection-log.json`） — 次サイクルの重複通知抑制に使用

---

## Completion Message

S4 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — S4: Change Detection 完了

検出結果（合計: X件）:
- 停滞商談: [X]件
- 未返信メール: [X]件
- 会議準備: [X]件
- 金額変動: [X]件
- 期限接近: [X]件（うちP1緊急: [Y]件）
- 未追跡リード: [X]件
- ステータス未反映: [X]件
- 矛盾フラグ（人間確認依頼）: [X]件

重複通知抑制で除外: [X]件
1商談1アクション原則により統合: [X]件

次のステップ:
- 対応要案件あり（X件）→ A1（Deal Health Scoring）へ続行
- 検出なし（0件）→ R1（Daily Summary Generation）へジャンプ

### WHAT'S NEXT
**A) 修正依頼** — 検出ルールの閾値や判定ロジックを修正する
**B) [A1 Deal Health Scoring / R1 Daily Summary Generation]へ続行** — 検出結果に基づいて続行する
```

---

## Error Handling

### エラー 1: Salesforce単独データによる停滞誤検出（Pitfall 1対策）

**症状**: Salesforceのステージ滞留日数のみで停滞を判定し、Slackで活発に交渉中の商談に停滞アラートを送信する。担当者が「毎日交渉しているのになぜ停滞アラートが来るのか」と不満を示す。
**原因**: Step 2の多層判定条件（条件B: Gmail/Slackアクティビティ確認）を省略している。
**対処**: 停滞検出は必ずAND条件（Salesforce停滞 **かつ** 他ソースでもアクティビティなし）を適用する。

**GOOD例**:
```
商談A: Proposalステージ16日（閾値14日超過）→ 条件A: 満たす
→ Gmail/Slack確認（7日以内）: Slackに「2026-04-04 価格交渉中」メッセージあり
→ 条件B: 満たさない（アクティビティあり）
→ conflict_stale_but_active フラグ（S3で付与済み）により除外
→ 「人間確認依頼」として登録（Pitfall 1・5対策）
```

**BAD例**:
```
商談A: Proposalステージ16日（閾値14日超過）
→ Salesforceのみで判定: 停滞商談として検出
→ 担当者に「商談Aが16日間停滞」とアラート送信
→ 担当者: 「毎日Slackで交渉しているのに停滞？エージェントは役に立たない」
→ エージェントへの信頼失墜（Pitfall 1対策漏れ）
```

### エラー 2: 同一商談への重複アラート送信（Pitfall 3対策）

**症状**: 商談Bに「停滞」「未返信」「期限接近」の3つの検出結果を個別に通知し、担当者が同日に3件の通知を受け取る。
**原因**: Step 9の1商談1アクション原則が適用されていない。
**対処**: 同一 `opportunity_id` の検出結果は必ず1件に統合し、追加情報はコンテキストとして付加する。

**GOOD例**:
```
商談B: 停滞（14日）+ 未返信（3日）+ 期限接近（5日）の3検出
→ 優先度順: deadline_approaching（P2）> stale_deal > unanswered_email
→ 1件に統合: 「期限接近（5日）」をメイン、他2件をコンテキストに付加
→ 担当者への通知: 1件（Pitfall 3対策）
```

**BAD例**:
```
商談B: 停滞・未返信・期限接近 を個別に3件通知
→ 担当者が同日に3件のSlack通知を受信
→ 重要な通知が大量通知に埋もれて見落とし
→ 翌日の期限接近に対応できず商談を逸失（Pitfall 3対策漏れ）
```

### エラー 3: 重複通知抑制の未適用（Pitfall 3対策）

**症状**: 前回巡回（1時間前）で送信した「未返信リマインダー」が、次回巡回でも再度送信される。担当者が1日に24件の同一リマインダーを受け取る。
**原因**: Step 3の前回検出ログとの照合処理（4時間抑制ルール）が未実装。
**対処**: `state/last-detection-log.json` を参照し、同一 `thread_id` または `opportunity_id` の通知が4時間以内に送信済みかを確認する。

**GOOD例**:
```
未返信メール（thread_def456）: 前回通知 2026-04-05T09:00
→ 現在時刻 2026-04-05T10:00（1時間後）
→ 4時間抑制ルール: (10:00 - 09:00) = 1時間 < 4時間
→ スキップ（重複通知として除外）
→ 次の有効通知タイミング: 2026-04-05T13:00以降
```

**BAD例**:
```
未返信メール（thread_def456）: 前回通知確認なし
→ 1時間ごとの巡回で毎回「未返信リマインダー」を送信
→ 担当者が1日に24件のリマインダーを受信
→ 通知を全て無視するようになる（Pitfall 3対策漏れ）
```

### エラー 4: フリーメールドメインのリード誤検出

**症状**: 社内で使用しているGmail（@gmail.com）からの内部テストメールが「未追跡リード」として検出され、毎サイクルにリード登録提案が発生する。
**原因**: Step 7の除外ドメインリスト（`config/excluded-domains.yaml`）の設定漏れ。
**対処**: 除外ドメインリストに主要フリーメールドメイン（gmail.com、yahoo.co.jp、hotmail.com等）と社内ドメインを事前登録する。新たな誤検出パターンが発見されたら除外リストを更新する。

**GOOD例**:
```
Gmail受信: from@gmail.com（個人アドレス）からの問い合わせ
→ 除外ドメインチェック: gmail.com は excluded-domains.yaml に登録済み
→ 未追跡リード検出をスキップ
→ 誤検出なし
```

**BAD例**:
```
Gmail受信: internal-test@gmail.com（社内テストアカウント）からのメール
→ 除外ドメインリストにgmail.comが未登録
→ 「未追跡リード: gmail.com」として検出
→ 毎サイクルに「新規リード登録提案」が発生
→ 担当者が毎時間無意味な承認リクエストを受け取る（Pitfall 3対策漏れ）
```
