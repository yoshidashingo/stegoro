# Data Source Config — 4ソース接続設定・OAuth管理・APIレート制限・フォールバック設定

## 基本方針

このファイルは Pitfall 4（API制限への対応不足）と Pitfall 5（データソース間の不整合）対策の中核です。
4データソースの接続設定・認証管理・レート制限対応・障害時フォールバックを一元管理します。

Salesforceは商談データのマスターソースとして扱い、他の3ソースは補助情報として位置づけます。

---

## 1. Salesforce 接続設定

### 認証設定

```yaml
salesforce:
  auth:
    method: "OAuth 2.0 Connected App"
    grant_type: "refresh_token"
    scopes:
      - "api"
      - "refresh_token"
      - "offline_access"
    api_version: "v59.0"            # Bulk API 2.0 対応バージョン以上

  endpoints:
    login: "https://login.salesforce.com/services/oauth2/token"
    instance: "{instance_url}"      # 認証後に動的取得
    api_base: "{instance_url}/services/data/v59.0"
    bulk_api: "{instance_url}/services/data/v59.0/jobs/ingest"
```

### ガバナーリミット管理（Pitfall 4対策）

```yaml
  governor_limits:
    daily_api_calls:
      limit: 150000                  # Enterprise Edition デフォルト
      safe_mode_threshold: 0.80      # 80%でセーフモード移行
      readonly_threshold: 0.95       # 95%で読み取り専用モード移行
      alert_threshold: 0.80          # 80%超で管理者にSlack通知

    soql_query_rows:
      limit: 50000                   # 1クエリあたり
      recommended_limit: 200         # 1クエリで取得する推奨行数

    dml_operations:
      limit: 150                     # 1トランザクションあたり

    concurrent_api_requests:
      limit: 10
```

### API使用量モニタリング

```yaml
  monitoring:
    check_interval: "cycle_start"    # 毎サイクル開始時に確認
    endpoint: "/limits"              # GET /services/data/v59.0/limits
    mode_transitions:
      normal_to_safe:
        condition: "usage_rate >= 0.80"
        action: "extend_cycle_interval_to_2h, prefer_bulk_api"
      safe_to_readonly:
        condition: "usage_rate >= 0.95"
        action: "stop_dml_operations, read_only"
      readonly_to_stop:
        condition: "usage_rate >= 1.00"
        action: "stop_all_operations, alert_admin"
```

### クエリ最適化ルール

```yaml
  query_rules:
    - rule: "差分取得を使用する"
      pattern: "WHERE LastModifiedDate >= :lastSyncTimestamp"
    - rule: "必要フィールドのみSELECT"
      forbidden: "SELECT * FROM ..."
    - rule: "LIMIT句を必ず付ける"
      recommended_limit: 200
    - rule: "大量レコードはBulk API 2.0を使用"
      threshold: 50
```

### リトライ設定（指数バックオフ）

```yaml
  retry:
    max_attempts: 5
    backoff_seconds: [1, 2, 4, 8, 16]
    retryable_errors:
      - "REQUEST_LIMIT_EXCEEDED"
      - "UNABLE_TO_LOCK_ROW"
      - "SERVER_ERROR"              # HTTP 500
    non_retryable_errors:
      - "INVALID_SESSION_ID"        # → トークンリフレッシュへ
      - "MALFORMED_QUERY"           # → クエリ修正が必要
```

---

## 2. Gmail 接続設定

### 認証設定

```yaml
gmail:
  auth:
    method: "Google OAuth 2.0"
    scopes:
      read_only: "https://www.googleapis.com/auth/gmail.readonly"
      send: "https://www.googleapis.com/auth/gmail.send"
    note: "送信スコープ(gmail.send)は承認済みアクション実行時のみ使用。
           通常巡回はgmail.readonlyのみ使用する"

  endpoints:
    token: "https://oauth2.googleapis.com/token"
    api_base: "https://gmail.googleapis.com/gmail/v1/users/me"
```

### レート制限

```yaml
  rate_limits:
    quota_units_per_second: 250     # 1ユーザーあたり
    quota_units_per_day: 1000000
    optimization: "バッチリクエストでAPIコールを集約する"
```

### データ取得設定

```yaml
  data_fetch:
    sync_strategy: "incremental"    # 差分取得（前回タイムスタンプ以降）
    fields:
      - "id"
      - "threadId"
      - "subject"
      - "from"
      - "to"
      - "date"
      - "snippet"                   # 本文全文ではなくスニペットのみ取得
    exclusion_filters:
      - "社内メールの除外フィルター（設定可能）"
      - "ノイズフィルター（ニュースレター、自動返信等）"
```

---

## 3. Google Calendar 接続設定

### 認証設定

```yaml
google_calendar:
  auth:
    method: "Google OAuth 2.0"
    scopes:
      - "https://www.googleapis.com/auth/calendar.readonly"

  endpoints:
    token: "https://oauth2.googleapis.com/token"
    api_base: "https://www.googleapis.com/calendar/v3"
```

### レート制限

```yaml
  rate_limits:
    queries_per_day: 1000000        # ユーザーあたり
```

### データ取得設定

```yaml
  data_fetch:
    time_range:
      past_days: 7
      future_days: 14
    filter:
      external_attendees_only: true # 外部参加者を含むミーティングのみ対象
    contact_matching:
      strategy: "email_domain_match"
      source: "SalesforceコンタクトのEmailフィールドと参加者メールを照合"
```

---

## 4. Slack 接続設定

### 認証設定

```yaml
slack:
  auth:
    method: "Slack OAuth 2.0 (Bot Token)"
    scopes:
      - "channels:read"
      - "channels:history"
      - "im:write"                  # DM送信（承認リクエスト・通知用）

  endpoints:
    api_base: "https://slack.com/api"
```

### 監視対象チャンネル設定

```yaml
  monitoring:
    channels:
      - "#sales-general"            # 設定ファイルで変更可能
      - "#deal-updates"
    keyword_filters:
      - "{商談名}"                  # 各商談名で動的に設定
      - "{顧客名}"
      - "{担当者名}"
    sync_strategy: "incremental"    # 前回タイムスタンプ以降のメッセージのみ

  notification:
    dm_targets: "担当者個人DM（承認リクエスト・リマインダー・緊急アラート）"
    channel_targets: "チームチャンネル（日次サマリー）"
```

---

## 5. トークンリフレッシュ管理

```yaml
token_management:
  check_timing: "S2: Source Connection（毎サイクル開始時）"
  refresh_threshold_minutes: 30    # 有効期限30分以内でリフレッシュ

  refresh_flow:
    - step: "アクセストークン有効期限確認"
    - step: "30分以内の場合: リフレッシュトークンでアクセストークンを更新"
    - step: "リフレッシュ成功: 新しいアクセストークンで処理続行"
    - step: "リフレッシュ失敗（リフレッシュトークン失効）:"
      action: "障害扱いとしてフォールバック処理へ移行。管理者にアラート送付"

  security:
    - "トークン情報は暗号化して保管する"
    - "監査ログへのトークン文字列の記録は禁止"
    - "トークンをSlackメッセージや通知に含めない"
```

---

## 6. フォールバック設定（Pitfall 4・Pitfall 5対策）

### ソース別フォールバック動作

```yaml
fallback:
  salesforce_down:
    action: "巡回中断。Gmail/Calendar/Slackのみで停滞外の検出は続行"
    승인제_actions: "停止（マスターデータソース不在のため）"
    report: "障害内容をレポートに明記"
    rationale: "Salesforceは商談データのマスターソース。障害時は承認制アクションを停止"

  gmail_down:
    action: "未返信メール検出をスキップ。他3ソースで巡回続行"
    skipped_detections:
      - "未返信メール検出"
      - "Gmail由来のActivity記録"

  calendar_down:
    action: "会議準備検出をスキップ。他3ソースで巡回続行"
    skipped_detections:
      - "ミーティング準備サマリー生成"
      - "Calendar由来のActivity記録"

  slack_down:
    action: "Slack由来のアクティビティ検出をスキップ"
    multilayer_judgment: "多層判定でSlackスコアを0として扱う（判定は継続）"

  all_sources_down:
    action: "巡回をスキップ。障害通知を緊急チャンネルに送付"
    notification: "管理者へ全ソース障害アラート"
```

### フォールバック優先度

```yaml
  source_priority:
    1: "Salesforce（商談データのマスターソース — 障害時は承認制アクションを停止）"
    2: "Gmail（未返信検出に必須 — 障害時は該当検出カテゴリをスキップ）"
    3: "Google Calendar（会議準備に必須 — 障害時は該当検出カテゴリをスキップ）"
    4: "Slack（多層判定の補助ソース — 障害時はスコア0で代替可能）"
```

---

## GOOD/BAD 例

**GOOD**: Salesforce APIの使用率が82%に達したため、エージェントが自動的にセーフモードに切り替えた。
巡回間隔を2時間に延長し、以降のSOQLクエリをBulk API 2.0に切り替えて必要フィールドのみ取得した。
管理者に「API使用率82%。セーフモードに移行しました」とSlack通知を送付した。
当日のガバナーリミット超過を回避し、夜間の日次サマリー生成まで正常に動作した（Pitfall 4対策）。

**BAD**: Google Calendar APIが障害中にもかかわらず、会議準備サマリーの生成を繰り返し試みた。
リトライがタイムアウトするまで巡回全体がブロックされ、他の3ソースの処理も停止した。
Calendar障害時は会議準備検出をスキップして他ソースの巡回を継続すること（フォールバック設定の不備）。
