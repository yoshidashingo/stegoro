# R2: Next-Day Prep — Phase Rule

## Purpose

夜間モード時に翌日の営業準備タスクを事前実行する。Google Calendarから翌営業日のミーティング予定を取得し、Salesforce・Gmail・Slackの情報を統合した会議準備サマリーを生成して担当者に送付する。翌日クロージング期限の商談アラートを事前準備し、翌朝8:05に送信するよう予約する。夜間の顧客向けアクションは全て禁止。

**Classification**: CONDITIONAL（夜間モード時）
**Approval Gate**: あり（CP-7: 翌日ミーティングサマリー・アラート準備完了の確認）
**参照元**: `workflow-architecture.md` R2 セクション

---

## Prerequisites

- S1で夜間モード（20:00-8:00 JST、または祝日）と判定されていること
- S2のソース接続状態が確定していること
- Google Calendarへの読み取りアクセスが可能であること
- Salesforce・Gmail・Slackへの読み取りアクセスが可能であること（書き込みは制限付き）
- 翌朝送信キュー（`state/morning-queue.yaml`）が参照可能であること

---

## Execution Classification

**CONDITIONAL EXECUTE**

- Execute IF: S1で **夜間モード** と判定された場合
- Skip IF: 日中モード時（このステージには到達しない）

---

## Execution Steps

### Step 1: 翌営業日のミーティング予定の取得

Google Calendarから翌営業日の外部参加者付きミーティング予定を取得する。

**翌営業日の算出**:
- 金曜夜間: 翌月曜日
- 祝日前夜: 祝日明けの最初の平日
- `config/japan-holidays.json` を参照して祝日を考慮

**Calendar API 呼び出し**:
```
calendar.events.list(
  timeMin = next_business_day + "T00:00:00+09:00",
  timeMax = next_business_day + "T23:59:59+09:00",
  singleEvents = true,
  orderBy = "startTime"
)
```

**フィルタリング**:
- 外部参加者（自社ドメイン以外）を含む予定のみ対象
- 終日イベント（all-day）は除外（商談ミーティングではない可能性が高い）
- S3と同様の商談紐づけ判定を適用

**取得件数が0件の場合**:
- ミーティングなしとして記録し、Step 2をスキップ
- 翌日クロージング期限チェック（Step 3）は続行

### Step 2: 会議準備サマリーの生成

各ミーティングに対して、Salesforce・Gmail・Slackの情報を統合した会議準備サマリーを生成する。

**サマリー生成のデータソース**:

| 情報源 | 取得内容 |
|--------|---------|
| Salesforce | 商談ステージ・金額・クロージング日・ヘルススコア・直近アクティビティ |
| Gmail | 直近3件のメール往復（件名・日付・概要） |
| Slack | 直近7日の商談関連メッセージ要約 |

**サマリーテンプレート**:
```markdown
## 会議準備サマリー — {event_summary}
**日時**: {event_start} / **場所/URL**: {event_location}
**参加者**: {attendees_list}

---

### 商談概要
- **商談名**: {opportunity_name}
- **金額**: ¥{amount} / **ステージ**: {stage}
- **クロージング予定**: {close_date}（{days_to_close}日後）
- **ヘルススコア**: {health_color}（{health_score}点）

### 直近のやり取り（過去14日）
{gmail_thread_summary}（最大3件）
{slack_mentions_summary}（最大5件）

### 前回ミーティングの主要論点
{previous_meeting_summary_if_available}

### 未解決の懸念事項
{open_concerns_detected_from_context}

### 推奨アジェンダ（エージェント提案）
- {suggested_agenda_item_1}
- {suggested_agenda_item_2}
- {suggested_agenda_item_3}

---
*このサマリーは営業パイプラインエージェントが自動生成しました。内容は参考情報です。*
```

**推奨アジェンダ生成ロジック**:
- ヘルススコアが赤の場合: 「現在の課題と解決方針の確認」を優先
- 未返信メールがある場合: 「{件名} についての確認」をアジェンダに追加
- 期限が近い場合: 「クロージングに向けた意思決定の確認」をアジェンダに追加

**配信方法**:
- 担当者Slack DM に翌朝8:05 送信予約（モーニングキューに追加）
- サマリー生成完了をサイクル状態ファイルに記録

### Step 3: 翌日クロージング期限アラートの事前準備

翌営業日または翌々営業日がクロージング日の商談を特定し、アラートを事前準備する。

**対象商談の特定**:
```
SELECT Id, Name, StageName, Amount, CloseDate, OwnerId
FROM Opportunity
WHERE CloseDate IN (next_business_day, next_business_day + 1)
  AND IsClosed = false
ORDER BY Amount DESC
```

**アラートコンテンツの生成**:
```
🔔【事前アラート】クロージング期限が近づいています

商談: {opportunity_name}
クロージング予定: {close_date}（{days_to_close}日後）
金額: ¥{amount}
ステージ: {stage} / ヘルススコア: {health_color}

推奨アクション:
- {推奨アクションリスト（A2のロジックを参照）}

Salesforceで確認: {sf_url}
```

**送信予約**:
- 翌朝8:05 にSlack DM送信予約（モーニングキューに追加）
- クロージング当日商談（days_to_close = 0または1）: P1として優先送信

### Step 4: 夜間制約の遵守確認

夜間モード中に禁止されているアクションが実行されていないかを確認する。

**夜間禁止アクション**:

| アクション | 禁止理由 |
|----------|---------|
| 顧客への外部メール送信 | 深夜の顧客連絡は信頼毀損 |
| 承認済み高リスクアクションの実行 | 夜間の商談データ変更は翌朝確認が困難 |
| 新規承認リクエストの送信（高リスク） | 担当者の深夜対応を強制しない |

**夜間許可アクション**:

| アクション | 許可理由 |
|----------|---------|
| データ読み取り（全ソース） | データ収集は副作用なし |
| 会議準備サマリーの**生成**（担当者宛） | 社内向け・翌朝8:05送信予約 |
| Salesforceへの低リスク書き込み（Activity記録のみ） | 書き込み範囲が限定的 |
| モーニングキューへの追加 | 実際の送信は翌朝8:05以降 |

**制約違反の検出**:
- 夜間モード中に `send_email`（顧客向け）実行を検出した場合: 即座にブロックし管理者通知

### Step 5: モーニングキューの確定

翌朝8:05に送信するコンテンツのキューを確定する。

```yaml
# state/morning-queue.yaml
morning_queue:
  execute_at: "2026-04-06T08:05:00+09:00"
  items:
    - type: "meeting_prep_summary"
      recipient_slack_id: "U123456789"
      event_summary: "株式会社A — 提案レビュー"
      event_start: "2026-04-06T10:00:00+09:00"
      content_file: "state/summaries/meeting-prep-cal-xyz789.md"

    - type: "deadline_alert"
      recipient_slack_id: "U987654321"
      opportunity_name: "株式会社H — Enterpriseプラン"
      close_date: "2026-04-06"
      priority: P1
      content: "{アラートメッセージ}"
```

---

## Output Artifacts

R2 が生成するアーティファクト:

1. **会議準備サマリーファイル**（`state/summaries/meeting-prep-{event_id}.md`） — 翌朝8:05送信予約
2. **翌日クロージング期限アラート** — モーニングキューに追加
3. **モーニングキュー**（`state/morning-queue.yaml`） — 翌朝8:05に D1相当の処理で自動実行

---

## Completion Message

R2 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — R2: Next-Day Prep 完了

翌日準備状況:
- 会議準備サマリー生成: [X]件（翌朝8:05に担当者DM送信予定）
- クロージング期限アラート: [X]件（翌朝8:05に担当者DM送信予定）
- 夜間禁止アクション実行: [なし / あり（異常 — 管理者通知済み）]

翌営業日: {next_business_date}

CP-7チェック: [PASS / 要確認]

### WHAT'S NEXT
**A) 修正依頼** — 翌日準備の内容や送信設定を修正する
**B) [R1 Daily Summary Generation]へ続行** — 日次サマリー生成へ進む
```

---

## Error Handling

### エラー 1: 夜間モードで顧客へのメールを即時送信（ドメイン固有）

**症状**: 夜間モード中に承認済みのフォローアップメールを即時実行した。顧客が深夜24:00にメールを受信し不快感を示した。
**原因**: Step 4の夜間制約チェックが実装されていない。
**対処**: R2の実行前に全実行アクションを夜間禁止リストと照合する。`send_email` の実行要求を検出した場合は即座にブロックし、`deferred_until: next_business_day_08:00` フラグを付与する。

**GOOD例**:
```
夜間モード（23:00）: send_email アクションの実行要求を検出
→ 夜間制約チェック: send_email = 禁止アクション
→ 実行ブロック
→ deferred_until = "2026-04-06T08:00:00+09:00"
→ モーニングキューに追加
→ 顧客への送信は翌朝8:00以降
```

**BAD例**:
```
夜間モード（23:00）: 夜間制約チェックなし
→ send_email 即時実行
→ 顧客が深夜にメールを受信
→ 「深夜にメールを送ってくる営業は非常識」と返信
→ 商談の信頼関係が毀損（夜間制約違反）
```

### エラー 2: 金曜夜間に翌営業日（月曜）でなく翌日（土曜）のカレンダーを参照

**症状**: 金曜夜間の巡回で翌日（土曜日）のカレンダーを参照し、予定が0件として会議準備サマリーが生成されない。月曜の重要ミーティングの準備が週末に実施されない。
**原因**: Step 1の翌営業日算出で曜日・祝日を考慮していない。
**対処**: 翌営業日の算出は必ず「土日・祝日を除いた次の平日」とする。金曜夜間は月曜日を対象とする。

**GOOD例**:
```
現在時刻: 金曜 21:00
→ 翌営業日算出: 土曜（スキップ）→ 日曜（スキップ）→ 月曜 = 翌営業日
→ 月曜のカレンダーを取得
→ 月曜10:00 「株式会社A — 週次レビュー」の会議準備サマリーを生成
→ 担当者が月曜朝8:05にサマリーを受信してコールドスタートなしに臨む
```

**BAD例**:
```
現在時刻: 金曜 21:00
→ 翌営業日算出: 単純に +1日 = 土曜
→ 土曜のカレンダーを取得: 予定0件
→ 会議準備サマリー生成なし
→ 月曜の重要ミーティングに向けた事前準備なし
→ 担当者が月曜朝に慌てて準備（自動化の恩恵なし）
```

### エラー 3: 会議準備サマリーにSalesforce情報のみを使用（Pitfall 5対策）

**症状**: 会議準備サマリーにSalesforce上の古い情報のみが含まれ、直近のGmail交渉内容・Slack議論が反映されない。担当者が前回交渉の重要な論点を把握せずにミーティングに臨む。
**原因**: Step 2のデータ統合処理でGmail・Slackを参照していない。
**対処**: サマリー生成時は必ず4ソース（Salesforce・Gmail・Calendar・Slack）の情報を統合する。各ソースの情報量が少ない場合は「情報なし」と明示する。

**GOOD例**:
```
会議準備サマリー生成:
→ Salesforce: Negotiationステージ、¥8M、クロージング4/15
→ Gmail直近3件: 4/3「価格交渉」4/4「追加要件」4/5「回答期限の確認」
→ Slack直近7日: 「4/5 担当者: 月曜に最終回答する予定」
→ 全情報を統合したサマリー
→ 担当者がGmail・Slackでの交渉経緯を把握してミーティングへ（Pitfall 5対策）
```

**BAD例**:
```
会議準備サマリー生成:
→ Salesforceのみ参照: Negotiationステージ、¥8M、LastActivityDate=3/20
→ 直近2週間のGmail交渉内容なし
→ 担当者: 「最近どんな交渉したんだっけ？」と Slackを慌てて検索
→ ミーティング開始5分前に過去メールを確認（自動化の恩恵なし）
```
