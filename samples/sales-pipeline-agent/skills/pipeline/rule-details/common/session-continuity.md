# Session Continuity — 巡回サイクル状態管理

## 基本原則

営業パイプライン管理エージェントのセッション単位は「巡回サイクル」です。
チケット単位ではなく、1時間ごとの巡回サイクルを基本単位として状態を管理します。
セッション状態は次のサイクル開始時に引き継がれ、繰り延べアクションや承認待ちキューが
失われないよう保証します。

---

## 巡回サイクル状態テンプレート

```yaml
cycle_state:
  # 基本情報
  cycle_id: "{YYYYMMDD_HHMMSS}"           # 例: 20240405_090000
  start_time: "{ISO8601タイムスタンプ}"
  mode: "daytime"                          # daytime | nighttime
  status: "running"                        # running | completed | timeout | error

  # ループ制御
  loop_control:
    current_loop: 1                        # 現在のループ回数（最大5）
    max_loops: 5
    elapsed_minutes: 0                     # 経過時間（分）
    timeout_minutes: 45                    # タイムアウト上限
    remaining_actions: 0                   # 未処理アクション件数

  # Salesforce API使用量
  api_usage:
    daily_limit: 150000                    # Enterprise Edition デフォルト
    used_today: 0
    used_this_cycle: 0
    usage_rate: 0.0                        # 使用率（0.0〜1.0）
    mode: "normal"                         # normal | safe_mode | readonly

  # ソース接続状態
  source_connections:
    salesforce:
      status: "connected"                  # connected | error | token_expired
      last_sync: "{ISO8601タイムスタンプ}"
      token_expires_at: "{ISO8601タイムスタンプ}"
    gmail:
      status: "connected"
      last_sync: "{ISO8601タイムスタンプ}"
      token_expires_at: "{ISO8601タイムスタンプ}"
    google_calendar:
      status: "connected"
      last_sync: "{ISO8601タイムスタンプ}"
      token_expires_at: "{ISO8601タイムスタンプ}"
    slack:
      status: "connected"
      last_sync: "{ISO8601タイムスタンプ}"
      token_expires_at: "{ISO8601タイムスタンプ}"

  # 承認待ちキュー
  pending_approvals:
    - approval_id: "{UUID}"
      opportunity_id: "{SalesforceOpportunityId}"
      opportunity_name: "{商談名}"
      action_type: "send_email"            # アクション種別
      requested_at: "{ISO8601タイムスタンプ}"
      reminder_sent: false
      timeout_at: "{ISO8601タイムスタンプ}"  # requested_at + 2時間
      status: "waiting"                    # waiting | approved | rejected | deferred

  # 繰り延べリスト（次サイクルへ引き継ぐアクション）
  deferred_actions:
    - action_id: "{UUID}"
      opportunity_id: "{SalesforceOpportunityId}"
      opportunity_name: "{商談名}"
      action_type: "{アクション種別}"
      original_cycle_id: "{最初に検出したサイクルID}"
      defer_count: 1                       # 繰り延べ回数
      defer_reason: "timeout"             # timeout | loop_limit | approval_timeout
      deferred_at: "{ISO8601タイムスタンプ}"

  # ロールバック用スナップショット
  rollback_snapshots:
    - snapshot_id: "{UUID}"
      opportunity_id: "{SalesforceOpportunityId}"
      action_type: "stage_update"
      before_state:
        StageName: "Qualification"
        Amount: 5000000
        CloseDate: "2024-04-30"
      executed_at: "{ISO8601タイムスタンプ}"
      rolled_back: false

  # 今サイクルの処理結果サマリー
  cycle_summary:
    detected_count: 0                      # 検出件数
    auto_executed_count: 0                 # 自動実行件数
    approval_requested_count: 0            # 承認リクエスト送付件数
    approved_count: 0                      # 承認済み実行件数
    rejected_count: 0                      # 却下件数
    deferred_count: 0                      # 繰り延べ件数
```

---

## 状態管理ルール

### サイクル開始時

1. 前サイクルの `deferred_actions` を読み込み、ASSESSMENTフェーズの先頭に挿入する
2. 前サイクルの `pending_approvals` で `status: waiting` のものを確認する
   - `timeout_at` を過ぎたものは `status: deferred` に更新して `deferred_actions` に移動する
3. `api_usage.used_today` を Salesforce API から取得して更新する
4. 各ソースのトークン有効期限を確認し、30分以内のものをリフレッシュする

### ループ中

1. 各ループ開始時に `loop_control.current_loop` をインクリメントする
2. 各ループ完了後に `loop_control.elapsed_minutes` を更新する
3. 条件チェック:
   - `current_loop >= max_loops` → PAACKAGINGフェーズへ
   - `elapsed_minutes >= timeout_minutes` → PAACKAGINGフェーズへ（タイムアウト）
   - `remaining_actions == 0` → 正常完了

### 承認待ち管理

```
承認リクエスト送付時:
  → pending_approvalsに追加
  → timeout_at = requested_at + 2時間 を設定

30分経過チェック（毎ループ）:
  → reminder_sent == false かつ 経過30分以上
  → Slack DMでリマインダー送信
  → reminder_sent = true に更新

2時間経過チェック（毎ループ）:
  → timeout_at を過ぎた承認
  → status = deferred に更新
  → deferred_actionsに移動（defer_reason: approval_timeout）
```

### ロールバック管理

自動実行するSalesforce書き込み操作の前に必ず以下を実行する:

1. 変更前の状態を `rollback_snapshots` に記録する
2. 操作実行後、問題があれば `rolled_back: true` に更新して前の状態に復元する
3. ロールバック完了後、担当者にSlack DM で通知する

---

## サイクル完了時の状態保存

```
完了時に保存するデータ:
  - deferred_actions（次サイクルへ引き継ぎ）
  - pending_approvals（status: waiting のもの）
  - rollback_snapshots（rolled_back: false のもの — 保持期間: 7日）
  - cycle_summary（監査ログとして永続保存）

削除するデータ:
  - 完了したロールバックスナップショット（rolled_back: true かつ 30日以上経過）
  - 古い承認記録（approved/rejected かつ 30日以上経過）
```

---

## GOOD/BAD 例

**GOOD**: 45分タイムアウトに達した際、残り3件のアクションを `deferred_actions` に記録し、
`defer_reason: timeout`、`defer_count: 1` を付与した。次サイクル開始時にこれらが
ASSESSMENTフェーズの先頭に挿入され、優先的に処理された。

**BAD**: サイクルタイムアウト時にセッション状態を保存せず終了した。次サイクルで承認待ちキューが
消失し、顧客へのフォローアップメール送信承認が失われた。重要な商談の対応が1日遅延した。
