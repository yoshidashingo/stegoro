# D1: Auto-Execute — Phase Rule

## Purpose

A4の自動実行リストに含まれる低リスクアクションを安全に実行する。同一Salesforceレコードへの並行更新を防ぐ排他制御、Bulk API 2.0を用いたAPI消費最小化、実行前スナップショット取得（D5ロールバック用）、個別アクション失敗が他に波及しない部分成功制御を実施し、実行完了記録を監査ログに残す。

**Classification**: CONDITIONAL（自動実行対象がある場合）
**Approval Gate**: なし（次ステージ D4 へ自動移行）
**参照元**: `workflow-architecture.md` D1 セクション

---

## Prerequisites

- A4の自動実行リスト（`state/auto-execute-list.yaml`）が確定していること
- S1のAPIモードが通常またはセーフモードであること（読み取り専用時はSalesforce書き込みをスキップ）
- D5のロールバック用スナップショット保存先（`state/pre-execution-snapshots/`）が存在すること

---

## Execution Classification

**CONDITIONAL EXECUTE**

- Execute IF: A4の自動実行リストにアクションが **1件以上** ある場合
- Skip IF: 自動実行対象が0件の場合 → D4（Loop Control）へ直行

---

## Execution Steps

### Step 1: 排他制御ロックの取得

同一OpportunityIDへの並行更新を防ぐため、実行前にレコードレベルのロックを取得する。

**ロック取得手順**:
```
FOR each action IN auto_execute_list:
  lock_key = "sf_opportunity_{action.target_opportunity_id}"

  IF lock_exists(lock_key):
    # 他のアクションがこのレコードを処理中
    action.status = "waiting"
    wait_max = 30秒
    retry_interval = 5秒
    retry_until_lock_released(lock_key, wait_max)

    IF still_locked after wait_max:
      action.status = "deferred"
      action.deferred_reason = "lock_timeout"
      move_to_deferred_list(action)
      CONTINUE  # 次のアクションへ

  ELSE:
    acquire_lock(lock_key, ttl=120秒)  # 最大2分間のロック
    action.lock_acquired = true
```

**ロックのライフタイム**:
- TTL（Time To Live）: 120秒。異常終了時でもロックが残り続けないよう上限を設定
- 実行完了後は必ず `release_lock(lock_key)` を呼び出す

### Step 2: 実行前スナップショットの取得（D5ロールバック用）

Salesforceレコードを変更するアクションの実行前に、現在の状態をスナップショットとして保存する。

**スナップショット取得対象**: `record_activity` および `update_stage_forward`
**スナップショット不要**: `send_reminder`、`generate_meeting_prep`（Salesforce書き込みを伴わない）

```yaml
# state/pre-execution-snapshots/{action_id}.yaml
snapshot:
  action_id: "ACT-20260405-001"
  captured_at: "2026-04-05T10:10:00+09:00"
  opportunity_id: "006xxxxxxxxxxxx"
  pre_state:
    StageName: "Proposal"
    Amount: 5000000
    LastActivityDate: "2026-03-20"
    activities_count: 12
```

### Step 3: Salesforceアクションの実行（Bulk API最適化）

**アクション種別ごとの実行方法**:

#### record_activity（Activity自動記録）

複数のActivity記録は Bulk API 2.0 でまとめて送信する。

**個別SOQL呼び出し禁止**: 5件以上のActivity記録は必ずBulk APIを使用。

```
# Bulk API 2.0 ジョブ構成
POST /services/data/v58.0/jobs/ingest
{
  "operation": "insert",
  "object": "Task",
  "contentType": "CSV"
}

# CSV本文（1リクエストで複数Task）
Subject,WhatId,ActivityDate,Status,Description
"[エージェント記録] Gmail送受信検出 2026-04-05","006xxx","2026-04-05","Completed","提案書送付メールをGmailで確認"
"[エージェント記録] Slackメッセージ検出 2026-04-04","006yyy","2026-04-04","Completed","価格交渉の言及をSlackで確認"
```

**Activityレコードのテンプレート**:
```
Subject: "[エージェント記録] {アクティビティ種別} {日時}"
Description: "{ソース}: {メール件名 or Slack要約 or Calendar件名}"
ActivityDate: {アクティビティ発生日}
Status: Completed
OwnerId: {担当者ID}
WhatId: {対象OpportunityID}
```

#### update_stage_forward（ステージ前進更新）

単件更新。Bulk APIは使用せずSOQL UPDATEで実行。

```
PATCH /services/data/v58.0/sobjects/Opportunity/{opportunity_id}
{
  "StageName": "{新ステージ名}"
}
```

#### send_reminder（担当者Slack DM）

Slack API `chat.postMessage` で送信。Salesforce APIを消費しない。

```json
{
  "channel": "{担当者Slack ID}",
  "text": "{リマインダーメッセージ}",
  "blocks": [...]
}
```

#### generate_meeting_prep（会議準備サマリー送付）

A2で生成した会議準備サマリーを担当者Slack DM に送付。Salesforce APIを消費しない。

### Step 4: 部分成功制御

個別アクションの失敗は他のアクション実行に影響させない。

**エラー処理ルール**:
```
FOR each action IN auto_execute_list:
  TRY:
    execute(action)
    action.status = "completed"
    record_execution_log(action, result="success")

  CATCH SalesforceAPIError as e:
    action.status = "failed"
    action.error_message = e.message
    record_execution_log(action, result="failed", error=e)
    release_lock(action.lock_key)  # ロック解放を必ず実施
    CONTINUE  # 次のアクションへ（中断しない）

  CATCH NetworkError as e:
    action.status = "failed"
    # 1回のみリトライ（Pitfall 4: 過剰リトライ防止）
    retry_once(action)
    IF retry_failed:
      move_to_deferred_list(action)
      CONTINUE

  FINALLY:
    release_lock(action.lock_key)  # 成功・失敗問わず必ずロック解放
```

**失敗しても継続するケース**:
- 個別ActivityのAPI呼び出し失敗: 他Activityの記録を継続
- Slack通知失敗: メールフォールバックを試行。それも失敗した場合はログに記録して継続
- ステージ更新失敗: 失敗アクションを繰り延べリストに移動して継続

**即座に停止するケース**:
- 認証エラー（401）: S2の接続状態を更新し、当該ソースの残アクションを全て繰り延べ
- 読み取り専用モードへの移行（API使用率95%超過を実行中に検出）: Salesforce書き込み系アクションを全て停止

### Step 5: 実行完了記録

全アクションの実行後、監査ログに記録する。

```yaml
execution_log:
  - action_id: "ACT-20260405-001"
    action_type: "record_activity"
    target_opportunity_id: "006xxxxxxxxxxxx"
    executor: "auto"
    executed_at: "2026-04-05T10:11:00+09:00"
    result: "completed"
    sf_record_id: "00Txxxxxxxxxxxx"  # 作成されたTaskのID

  - action_id: "ACT-20260405-004"
    action_type: "send_reminder"
    target_opportunity_id: "006yyyyyyyyyyyyy"
    executor: "auto"
    executed_at: "2026-04-05T10:11:05+09:00"
    result: "completed"
    slack_message_ts: "1680000000.123456"
```

**サイクル状態ファイルへの追記**:
```markdown
## Action Queue
- Auto-Execute: [総件数]件 / Completed: [成功件数]件 / Failed: [失敗件数]件
- [timestamp] D1: Auto-executed [X] actions
```

---

## Output Artifacts

D1 が生成するアーティファクト:

1. **実行完了記録**（監査ログ追記） — D5・R3で参照
2. **実行前スナップショット**（`state/pre-execution-snapshots/`） — D5のロールバック判定に使用
3. **失敗・繰り延べアクションリスト更新** — D4のループ制御で参照

---

## Completion Message

D1 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — D1: Auto-Execute 完了

実行結果:
- 成功: [X]件（Activity記録: [X]件、リマインダー: [X]件、会議準備: [X]件）
- 失敗（繰り延べ）: [X]件
- ロック競合で繰り延べ: [X]件

### WHAT'S NEXT
**A) 修正依頼** — 実行結果や繰り延べの扱いを修正する
**B) [D4 Loop Control]へ続行** — 残件チェックとループ判定へ進む
```

---

## Error Handling

### エラー 1: 同一商談への並行更新によるデータ競合（ドメイン固有）

**症状**: 同一商談に対してSalesforce ActivityをAPIで記録中に、別のアクション（ステージ更新）が同じ商談のフィールドを更新し、Salesforceのロック競合エラー（UNABLE_TO_LOCK_ROW）が発生する。
**原因**: Step 1の排他制御ロック取得を省略した実装。
**対処**: 同一 `opportunity_id` を持つアクションは、前のアクションのロック解放を確認してから次のアクションを実行する。

**GOOD例**:
```
ACT-001（record_activity, opportunity: 006xxx）とACT-005（update_stage, opportunity: 006xxx）が同時実行候補
→ ACT-001がlock_key "sf_opportunity_006xxx" を取得して実行開始
→ ACT-005: lock_exists = true → 待機（最大30秒）
→ ACT-001完了 → release_lock("sf_opportunity_006xxx")
→ ACT-005がロック取得 → 順次実行
→ UNABLE_TO_LOCK_ROW エラーなし
```

**BAD例**:
```
ACT-001とACT-005が並行実行（排他制御なし）
→ 両方が同時に sf_opportunity_006xxx レコードにアクセス
→ Salesforce: UNABLE_TO_LOCK_ROW エラー発生
→ 一方のアクションが失敗、データが部分的に更新された不整合状態に
→ D5のロールバックチェックで検出されるまで気づかない
```

### エラー 2: 個別API呼び出しによるガバナーリミット消費（Pitfall 4対策）

**症状**: 5件のActivity自動記録を個別のAPI呼び出し5回で実行し、必要以上のガバナーリミットを消費する。
**原因**: Step 3のBulk API使用条件チェックが実装されていない。
**対処**: 2件以上のActivity記録は必ずBulk API 2.0にまとめて送信する。個別送信は1件のみの場合に限定する。

**GOOD例**:
```
Activity記録アクション: 5件
→ Bulk API 2.0 ジョブ作成 → CSV 5行を1リクエストで送信
→ API呼び出し消費: 3回（ジョブ作成・データ送信・結果取得）
→ 個別送信5回より 40% API消費を削減（Pitfall 4対策）
```

**BAD例**:
```
Activity記録アクション: 5件
→ 個別SOQL INSERT × 5回
→ API呼び出し消費: 5回
→ 1巡回で50件のActivity記録が必要な場合: API呼び出し50回を消費
→ ガバナーリミットを不要に消費、他業務への影響リスク（Pitfall 4対策漏れ）
```

### エラー 3: ロック未解放による後続アクションのデッドロック

**症状**: D1実行中に予期しない例外が発生し、ロックが解放されないままD4・次サイクルに進んでしまう。次サイクルで同じ商談のアクションがロック待ちでタイムアウトする。
**原因**: Step 1のfinally句でのロック解放が実装されていない。
**対処**: try/finally 構造でエラー発生時も `release_lock()` を必ず呼び出す。TTL（120秒）による自動期限切れをバックアップとして設定する。

**GOOD例**:
```
ACT-003実行中にNetworkError発生
→ finally: release_lock("sf_opportunity_006zzz")
→ ロックが正常解放
→ 次のアクションが待機なしでロックを取得可能
→ TTL（120秒）も念のため設定済み（バックアップ）
```

**BAD例**:
```
ACT-003実行中にNetworkError発生
→ except節のみで処理（finally なし）
→ release_lock が呼ばれない
→ ロックが残存（TTL未設定のため永久にロック）
→ 次サイクルで "sf_opportunity_006zzz" へのアクション全てが30秒待機してタイムアウト
→ デッドロック状態で複数サイクルが機能不全に
```

### エラー 4: 読み取り専用モード移行後のDML実行（Pitfall 4対策）

**症状**: D1実行中にSalesforce API使用率が95%を超えて読み取り専用モードに移行したが、残りのActivity記録アクションをそのまま実行してDMLエラーが連続発生する。
**原因**: 実行中のAPIモード変化を監視していない。
**対処**: 各Salesforce書き込みアクションの実行前に現在のAPIモードを確認する。読み取り専用モードへの移行を検出した場合は残り全ての書き込みアクションを繰り延べリストに移動して即座に停止する。

**GOOD例**:
```
Activity記録 3件目実行前 → API使用率再確認: 96%（読み取り専用モードに移行）
→ 残り Activity記録 2件を繰り延べリストに移動
→ Salesforce書き込みを停止（Slack通知系は継続）
→ 管理者通知: 「API使用率96%: D1を中断し書き込みアクションを繰り延べました」
→ DMLエラー 0件（Pitfall 4対策）
```

**BAD例**:
```
D1実行中 API使用率監視なし
→ Activity記録 5件を順次実行中に使用率100%に到達
→ DMLエラーが連続発生
→ リトライにより使用率がさらに超過
→ Salesforce組織全体の API 枯渇（他業務システムにも影響）（Pitfall 4対策漏れ）
```
