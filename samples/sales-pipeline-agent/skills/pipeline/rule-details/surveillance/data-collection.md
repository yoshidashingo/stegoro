# S3: Data Collection — Phase Rule

## Purpose

S2で接続確認済みのソース（Salesforce / Gmail / Google Calendar / Slack）から差分データを収集し、統合ビューを構築する。前回巡回タイムスタンプ以降の変更データのみを取得（全量取得禁止）、ソース間矛盾を検出してフラグを付与し、S4の変更検出ロジックが利用する統合データセットを確定する。

**Classification**: ALWAYS EXECUTE（日中モード時に必須）
**Approval Gate**: なし（次ステージ S4 へ自動移行）
**参照元**: `workflow-architecture.md` S3 セクション

---

## Prerequisites

- S2のソース接続状態が確定していること（`source_status` が記録済み）
- S2のスキップカテゴリリストが参照可能であること
- 前回巡回タイムスタンプ（`state/last-collection-timestamp.json`）が存在すること
- S1のAPIモード（通常 / セーフ / 読み取り専用）が確定していること

---

## Execution Classification

**ALWAYS EXECUTE**（日中モード時）

- Execute IF: S2で少なくとも1ソース以上が `connected` 状態の場合
- Skip IF: S2で全ソース障害（このステージには到達しない）

---

## Execution Steps

### Step 1: 差分取得基準タイムスタンプの確定

`state/last-collection-timestamp.json` から前回巡回完了時刻を取得する。

```json
{
  "last_collection_completed_at": "2026-04-05T09:00:00+09:00",
  "per_source": {
    "salesforce": "2026-04-05T09:00:00+09:00",
    "gmail": "2026-04-05T09:00:00+09:00",
    "google_calendar": "2026-04-05T09:00:00+09:00",
    "slack": "2026-04-05T09:00:00+09:00"
  }
}
```

**タイムスタンプが存在しない場合**（初回実行または状態ファイル消失）:
- 現在時刻から24時間前を差分取得の起点とする
- 初回フラグを監査ログに記録

**前回ソース障害があった場合**:
- 障害ソースの `per_source` タイムスタンプは障害前の最終成功時刻を維持
- 障害解消後の初回収集では、最終成功時刻からの差分を取得（空白期間を補完）

### Step 2: Salesforceデータ収集

**収集対象レコードと項目**:

| オブジェクト | 収集条件 | 収集項目 |
|------------|---------|---------|
| Opportunity（商談） | `LastModifiedDate >= {前回タイムスタンプ}` | Id, Name, StageName, Amount, CloseDate, OwnerId, LastModifiedDate, LastActivityDate |
| Task / Event（Activity） | `CreatedDate >= {前回タイムスタンプ}` かつ `WhatId` がOpportunity | Id, WhatId, Subject, ActivityDate, Status, OwnerId, CreatedDate |
| Lead（リード） | `CreatedDate >= {前回タイムスタンプ}` または `LastModifiedDate >= {前回タイムスタンプ}` | Id, Email, Company, Status, OwnerId, ConvertedDate |

**Salesforce API呼び出し最適化（Pitfall 4対策）**:
- 取得件数 < 50件: SOQL APIを使用
- 取得件数 >= 50件: **Bulk API 2.0** を使用（個別SOQL呼び出し禁止）
- 1回のSOQLクエリで複数オブジェクトを結合せず、オブジェクト別に個別クエリを実行（ガバナーリミット内）
- セーフモード時: 取得件数上限を通常の50%に制限（API消費を半減）

**SOQLクエリ例（Opportunityの差分取得）**:
```sql
SELECT Id, Name, StageName, Amount, CloseDate, OwnerId,
       LastModifiedDate, LastActivityDate
FROM Opportunity
WHERE LastModifiedDate >= 2026-04-05T09:00:00+09:00
  AND IsClosed = false
ORDER BY LastModifiedDate DESC
LIMIT 200
```

### Step 3: Gmailデータ収集

**収集対象（S2でGmailが `connected` の場合のみ実行）**:

Gmail `messages.list` APIで以下の条件で検索:
```
query: "after:{前回タイムスタンプ Unix時刻} is:sent OR is:received"
```

**収集項目**:

| 項目 | 説明 | S4での用途 |
|------|------|----------|
| Message-Id | メール固有ID | 返信追跡・重複防止 |
| From / To / Cc | 送受信者メールアドレス | 顧客メールドメインとの照合 |
| Subject | 件名 | 商談名・顧客名とのマッチング |
| Date | 送受信日時（ISO 8601） | アクティビティ日時判定 |
| Thread-Id | スレッドID | 返信有無の確認 |
| Snippet | 本文の先頭200文字 | コンテキスト付与用 |

**収集除外条件**:
- 内部メールドメイン（`@{自社ドメイン}`）同士のやり取り
- スパム・プロモーションラベル付きメール
- 自動返信メール（`Auto-Reply`・`Out-of-Office` ヘッダー検出）

### Step 4: Google Calendarデータ収集

**収集対象（S2でGoogle Calendarが `connected` の場合のみ実行）**:

`calendar.events.list` APIで以下の条件で取得:
```
timeMin: {現在時刻}
timeMax: {現在時刻 + 14日}
singleEvents: true
orderBy: startTime
```

**収集項目と絞り込み条件**:

| 収集条件 | 収集項目 |
|---------|---------|
| 外部参加者（`@{自社ドメイン}` 以外）を含む予定のみ | Event-Id, Summary（件名）, Start, End, Attendees（メールアドレス）, HtmlLink |

**商談との紐づけ判定**:
1. 予定の件名または説明文に商談名・顧客社名が含まれているかを確認
2. 参加者メールアドレスがSalesforce Opportunityの関連取引先責任者と一致するかを確認
3. いずれかが一致する場合、`related_opportunity_id` として商談IDを付与

### Step 5: Slackデータ収集

**収集対象（S2でSlackが `connected` の場合のみ実行）**:

`conversations.history` APIで指定チャンネルの最新メッセージを取得:
```
channel: {config/data-source-config.md に定義されたチャンネルID一覧}
oldest: {前回巡回タイムスタンプ Unix時刻}
limit: 200
```

**収集対象チャンネル**（設定ファイル参照）:
- `#sales-pipeline`: 商談全般の更新情報
- `#deal-updates`: 個別商談スレッド
- `#{顧客名}-deal`: 顧客別チャンネル（動的チャンネル名に対応）

**収集項目とフィルタリング**:
- 収集: Timestamp, User, Text（メッセージ本文）, Channel
- フィルタ: 商談名・顧客社名・ドメイン名を含むメッセージのみ（全メッセージを保存しない）
- 個人情報（電話番号・メールアドレス等）をメッセージ本文から除去してから保存

### Step 6: 統合ビュー構築

収集した4ソースのデータをタイムスタンプベースで最新優先にマージし、商談ごとの統合ビューを構築する。

**統合ビューのデータ構造**:
```yaml
integrated_view:
  - opportunity_id: "006xxxxxxxxxxxx"
    opportunity_name: "株式会社A — Enterpriseプラン導入"
    stage: "Proposal"
    amount: 5000000
    close_date: "2026-04-30"
    owner_id: "005xxxxxxxxxxxx"
    last_salesforce_activity_date: "2026-03-20"
    days_in_stage: 16

    gmail_activities:
      - date: "2026-04-03"
        type: "sent"
        subject: "提案書ご送付の件"
        thread_id: "thread_abc123"

    calendar_events:
      - event_id: "cal_xyz789"
        summary: "株式会社A — 提案レビュー"
        start: "2026-04-06T10:00:00+09:00"
        related_opportunity_id: "006xxxxxxxxxxxx"

    slack_mentions:
      - timestamp: "2026-04-04T14:30:00+09:00"
        channel: "#deal-updates"
        text: "株式会社Aから価格交渉の連絡あり、来週回答予定"

    conflict_flags: []
    latest_activity_date: "2026-04-04"  # 全ソースの最新アクティビティ日
```

**タイムスタンプ優先ルール**:
- 同一情報が複数ソースで取得できる場合（例: Salesforce Activity + Gmail）、最新タイムスタンプのデータを正とする
- Salesforceのステージ情報はSalesforceを正とする（シングルソースオブトゥルース原則）

### Step 7: ソース間矛盾検出（Pitfall 5対策）

統合ビューに対して矛盾パターンを検出し、フラグを付与する。

**矛盾検出パターン**:

| 矛盾パターン | 検出条件 | 付与フラグ |
|------------|---------|----------|
| クローズ後の継続 | Salesforce Stage=「Closed Won/Lost」 かつ Gmail/Slackで直近7日以内に商談継続メッセージ | `conflict_post_close_activity` |
| 停滞判定矛盾 | Salesforce LastActivityDateが閾値超え かつ Gmail/Slackで直近N日以内にアクティビティあり | `conflict_stale_but_active` |
| ステータス未反映 | Gmail送受信またはCalendarミーティング実施 かつ Salesforce Activityに記録なし（24時間以上） | `conflict_activity_not_recorded` |
| 金額矛盾 | Salesforce Amount かつ Slackで「金額変更」「値引き」言及があるが Salesforceに反映なし | `conflict_amount_discrepancy` |

**矛盾フラグ付き案件の扱い**:
- S4で自動検出対象から除外（人間判断に委ねる）
- A2でアクション生成を保留し「人間確認依頼」アクションを生成
- R1サマリーの「要確認事項」セクションに記載

### Step 8: 収集完了後の後処理

1. **前回巡回タイムスタンプを更新**: 収集完了時刻を各ソースごとに記録
2. **統合ビューをサイクル状態ファイルに記録**: S4・A1・A2が参照可能にする
3. **収集件数サマリーをサイクル状態ファイルに追記**:
   ```markdown
   ## Data Collection Summary
   - Salesforce: Opportunities 12件, Activities 7件, Leads 3件
   - Gmail: Messages 18件
   - Google Calendar: Events 5件（商談関連: 3件）
   - Slack: Messages 24件（商談関連: 11件）
   - Conflict Flags: 2件
   ```

---

## Output Artifacts

S3 が生成するアーティファクト:

1. **統合ビュー**（`state/integrated-view.yaml`） — S4・A1・A2・A3で参照
2. **矛盾フラグリスト** — A2での「人間確認依頼」アクション生成に使用
3. **収集完了タイムスタンプ**（`state/last-collection-timestamp.json`） — 次サイクルの差分基準に使用
4. **収集件数サマリー** — R1のサマリーレポートに記載

---

## Completion Message

S3 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — S3: Data Collection 完了

収集結果:
- Salesforce: 商談 [X]件 / Activity [Y]件 / リード [Z]件
- Gmail: メッセージ [X]件
- Google Calendar: 予定 [X]件（商談関連: [Y]件）
- Slack: メッセージ [X]件（商談関連: [Y]件）

矛盾フラグ: [X]件
- [矛盾の種別と対象商談名のリスト（件数が多い場合は上位3件）]

スキップしたソース: [なし / ソース名と理由]

### WHAT'S NEXT
**A) 修正依頼** — 収集範囲や矛盾検出ルールを修正する
**B) [S4 Change Detection]へ続行** — 変更・異常の検出へ進む
```

---

## Error Handling

### エラー 1: 全量取得によるAPIガバナーリミット超過（Pitfall 4対策）

**症状**: 差分取得ではなく全量取得（`LastModifiedDate` フィルタなし）でSalesforceを照会し、数千件のレコードを取得してガバナーリミットを消費する。
**原因**: Step 1の前回タイムスタンプ確認を省略、またはSOQLクエリのWHERE句が欠落。
**対処**: 前回タイムスタンプがない場合でも必ず24時間前を基準に差分取得する。全量取得クエリをポリシーレベルで禁止する。

**GOOD例**:
```
前回タイムスタンプ: 2026-04-05T09:00:00+09:00
→ SOQL: WHERE LastModifiedDate >= 2026-04-05T09:00:00+09:00
→ 取得件数: Opportunity 12件（差分のみ）
→ API呼び出し消費: 1回
→ ガバナーリミットへの影響を最小化（Pitfall 4対策）
```

**BAD例**:
```
前回タイムスタンプが見つからない → WHERE句なしで全量取得
→ SOQL: SELECT * FROM Opportunity（フィルタなし）
→ 取得件数: 3,000件（全商談）
→ API呼び出し消費: 60回（50件/回 × 60回）
→ ガバナーリミットの60%を1回の巡回で消費
→ 同日他の業務システムのAPI呼び出しが制限に到達
```

### エラー 2: ソース間矛盾の無視による誤判定（Pitfall 5対策）

**症状**: Salesforceで停滞判定対象の商談について、Slackで担当者が顧客と活発にやり取り中にもかかわらず、矛盾フラグを付与せず停滞アラートを自動生成する。
**原因**: Step 7の矛盾検出処理が実装されていない、またはSlackデータを統合ビューに含めていない。
**対処**: 統合ビュー構築時に全ソースのアクティビティを集約し、`latest_activity_date` を全ソース横断で算出する。

**GOOD例**:
```
商談X: Salesforce LastActivityDate = 2026-03-20（17日前）→ 停滞候補
→ Slack #deal-updates で 2026-04-04 に「価格交渉中」メッセージを検出
→ conflict_stale_but_active フラグを付与
→ S4で自動停滞検出対象から除外
→ A2で「人間確認依頼」アクション生成（Pitfall 1・Pitfall 5対策）
```

**BAD例**:
```
商談X: Salesforce LastActivityDate = 2026-03-20（17日前）
→ Slackデータを統合ビューに含めずSalesforceのみで判定
→ S4で停滞商談として検出
→ 担当者に「商談Xが17日間停滞」とアラート送信
→ 担当者: 「毎日Slackで交渉しているのに何のアラートだ」と不満（Pitfall 1・5対策漏れ）
```

### エラー 3: 個人情報（PII）のSlackデータへの混入

**症状**: Slackメッセージに含まれる顧客の電話番号・個人メールアドレスが統合ビューにそのまま記録される。
**原因**: Step 5のPII除去処理が未実装。
**対処**: Slackメッセージ本文からPIIパターン（電話番号・メールアドレス・クレジットカード番号）を検出・マスクしてから統合ビューに保存する。

**GOOD例**:
```
Slackメッセージ: 「田中部長（090-xxxx-xxxx）から連絡あり、来週月曜に確認」
→ 電話番号パターン検出: 090-xxxx-xxxx
→ マスク後: 「田中部長（[TEL]）から連絡あり、来週月曜に確認」
→ マスク済みテキストのみ統合ビューに保存
→ 個人情報の不要な記録を防止
```

**BAD例**:
```
Slackメッセージ: 「田中部長（090-1234-5678）から連絡、メール: tanaka@client.com」
→ PIIチェックなしでそのまま統合ビューに保存
→ state/integrated-view.yaml に顧客の個人情報が平文で記録
→ 監査時にSOC 2準拠違反として指摘（データセキュリティリスク）
```

### エラー 4: Calendar予定と商談の紐づけ漏れ

**症状**: 翌日に重要な顧客ミーティングがあるが、Step 4の紐づけ判定で商談IDが付与されず、S4の「会議準備」検出で漏れる。
**原因**: 予定の件名が商談名と完全一致しないため（例: 商談名「株式会社A Enterpriseプラン」、予定件名「A社 定例MTG」）。
**対処**: 完全一致だけでなく、顧客社名の部分一致（前方一致・後方一致）と参加者メールドメインによる照合も実施する。紐づけ精度向上のため複数条件のOR判定を適用する。

**GOOD例**:
```
Calendar予定: "A社 定例MTG" / 参加者: tanaka@client-a.com
→ 件名の完全一致: なし
→ 参加者ドメイン client-a.com ↔ Salesforce取引先 client-a.com: 一致
→ related_opportunity_id を商談「株式会社A Enterpriseプラン」に設定
→ S4の「会議準備」検出に正しく含まれる
```

**BAD例**:
```
Calendar予定: "A社 定例MTG" / 参加者: tanaka@client-a.com
→ 件名完全一致のみでチェック: "株式会社A Enterpriseプラン" ≠ "A社 定例MTG"
→ 紐づけなし（related_opportunity_id = null）
→ S4で会議準備検出から漏れ
→ 担当者が直前に気づいて会議準備を手動で実施（自動化の恩恵なし）
```
