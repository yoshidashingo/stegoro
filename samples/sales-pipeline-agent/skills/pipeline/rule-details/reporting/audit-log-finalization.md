# R3: Audit Log Finalization — Phase Rule

## Purpose

本サイクルの全アクション履歴・承認記録・ロールバック記録・ソース接続状態・API使用量を確定して保存する。監査ログの完全性チェック（記録されたアクション数と実際の実行数の照合）を行い、保持期間ポリシーに基づいた期限超過ログのパージ対象フラグ付けを実施する。サイクルを正常完了状態として記録する。

**Classification**: ALWAYS EXECUTE（全サイクルで必須）
**Approval Gate**: なし（次回巡回まで待機）
**参照元**: `workflow-architecture.md` R3 セクション

---

## Prerequisites

- R1の日次サマリー配信が完了していること
- R2の翌日準備が完了していること（夜間モード時のみ）
- D1・D3の実行完了記録が参照可能であること
- D5のロールバック検証結果が参照可能であること
- 監査ログストレージ（`logs/audit/`）への書き込み権限があること

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: 常に実行（全サイクルの最終ステージ）
- Skip IF: なし（スキップ不可）

---

## Execution Steps

### Step 1: 本サイクルの全記録の集約

サイクル状態ファイル・各ステージの出力アーティファクトを集約して、サイクルサマリーを作成する。

**集約対象**:

| カテゴリ | 集約元 |
|---------|--------|
| 自動実行アクション一覧 | D1実行ログ |
| 承認リクエスト一覧 | D2・D3の処理記録 |
| 次サイクル繰り延べリスト | D4・D3の繰り延べ記録 |
| ロールバック記録 | D5の検証結果 |
| ソース接続状態 | S2の接続記録 |
| API使用量記録 | S1・D1のAPI使用量 |
| 矛盾フラグ案件 | S3の矛盾検出結果 |

**サイクルサマリーのフォーマット**:
```yaml
cycle_summary:
  cycle_id: "cycle-uuid-xxxx"
  started_at: "2026-04-05T10:00:00+09:00"
  completed_at: "2026-04-05T10:52:00+09:00"
  mode: "daytime"
  loop_count: 2
  exit_reason: "all_actions_completed"  # all_actions_completed / loop_limit / timeout

  source_status:
    salesforce: connected
    gmail: connected
    google_calendar: connected
    slack: connected

  api_usage:
    salesforce_daily_rate: "48%"
    bulk_api_jobs: 3
    dml_rows: 12

  detection_summary:
    total_detected: 7
    stale_deals: 2
    unanswered_emails: 1
    meeting_prep: 2
    deadline_approaching: 1
    status_not_recorded: 1
    conflict_flags: 1

  action_summary:
    auto_executed: 5
    auto_failed: 0
    approved: 3
    rejected: 1
    deferred_loop_limit: 0
    deferred_approval_timeout: 0
    deferred_next_cycle: 2

  rollback_summary:
    anomalies_detected: 0
    rollbacks_executed: 0

  notifications_sent:
    urgent: 0
    summary: 1
    individual_dm: 3
```

### Step 2: 監査ログエントリの確定

本サイクルの全アクションを ISO 8601 タイムスタンプ付きの append-only 形式で監査ログに書き込む。

**監査ログエントリフォーマット**:
```
[cycle_id] [timestamp] [action_type] [target_record_id] [executor: auto/human/system]
[result: executed/approved/rejected/deferred/rolled_back/skipped]
[approver: 担当者名（承認制の場合）]
[notes: 承認理由・却下理由・ロールバック理由・スキップ理由]
```

**監査ログ出力例**:
```
cycle-uuid-xxxx 2026-04-05T10:11:00+09:00 record_activity 006xxxxxxxxxxxx auto
  result: executed
  notes: Gmail送受信検出（4/3）をSalesforce Activityに自動記録

cycle-uuid-xxxx 2026-04-05T10:26:00+09:00 send_email 006yyyyyyyyyyyyy human
  result: approved
  approver: 田中太郎（U123456789）
  notes: フォローアップメール送信。顧客yamada@client-h.co.jpへ送信完了

cycle-uuid-xxxx 2026-04-05T10:25:00+09:00 update_stage_backward 006zzzzzzzzzzzzz human
  result: rejected
  approver: 田中太郎（U123456789）
  notes: 却下理由: 顧客との交渉で既にステージ変更を保留（context_mismatch）

cycle-uuid-xxxx 2026-04-05T10:45:00+09:00 send_reminder 006aaaaaaaaaaaaaa auto
  result: deferred
  notes: ループ上限5回到達のため次サイクルへ繰り延べ
```

**監査ログの保存先**:
- `logs/audit/{YYYY-MM}/audit-{YYYY-MM-DD}.log`（日次ファイル）
- **Salesforce外部に保存**（監査ログ自体がSalesforce APIを消費しないよう）
- **append-only**（既存ログの上書き・削除禁止）

### Step 3: 完全性チェック

記録されたアクション数と実際の実行数が一致することを確認する。

**チェック項目**:
```
# 実行数の照合
recorded_auto_executed = count(audit_log WHERE result == 'executed' AND executor == 'auto')
actual_auto_executed = D1.completed_count

IF recorded_auto_executed != actual_auto_executed:
  flag_integrity_error("自動実行記録数不一致",
                       expected=actual_auto_executed,
                       recorded=recorded_auto_executed)

# 承認数の照合
recorded_approved = count(audit_log WHERE result == 'approved')
actual_approved = D3.approved_count

IF recorded_approved != actual_approved:
  flag_integrity_error("承認記録数不一致",
                       expected=actual_approved,
                       recorded=recorded_approved)
```

**許容差分**: 0（完全一致が原則）
**差分が検出された場合**: 管理者に通知し、不整合の詳細を監査ログに追記する

### Step 4: 保持期間ポリシーの適用

`config/action-authority.md` または `config/retention-policy.yaml` に定義された保持期間ポリシーに従い、期限超過ログをパージ対象としてフラグ付けする。

**デフォルト保持期間**:

| ログ種別 | 保持期間 |
|---------|---------|
| 監査ログ（全アクション記録） | 12ヶ月 |
| サイクル状態ファイル | 30日 |
| 実行前スナップショット | 7日（ロールバック後は30日） |
| 通知ログ | 14日 |
| 繰り延べアクションリスト | 次サイクル実行後7日 |

**パージ対象フラグ付け**（実際の削除は別プロセス）:
```
FOR each log_file IN log_directory:
  IF log_file.created_at < (today - retention_period):
    mark_for_purge(log_file)
    log("パージ対象フラグ: {log_file.name} (作成: {log_file.created_at})")
```

### Step 5: サイクル完了状態の記録

サイクル状態ファイルを「完了」として更新し、次回巡回に向けた状態を確定する。

```yaml
# state/current-cycle.yaml の最終更新
cycle_status: "completed"
completed_at: "2026-04-05T10:52:00+09:00"
next_cycle_estimated_at: "2026-04-05T11:00:00+09:00"

# 次サイクルへの引き継ぎ情報
deferred_actions_count: 2
suspended_action_types: []
api_usage_at_completion: "48%"
```

**状態の確定により**、次サイクルのS1がこのファイルを参照して繰り延べリストの引き継ぎを正しく実施できる。

---

## Output Artifacts

R3 が生成するアーティファクト:

1. **確定済み監査ログ**（`logs/audit/{YYYY-MM}/audit-{YYYY-MM-DD}.log`） — コンプライアンス証跡
2. **サイクルサマリー**（`logs/cycle-summaries/{cycle_id}.yaml`） — 週次パイプラインレビュー素材
3. **サイクル完了状態**（`state/current-cycle.yaml`） — 次サイクルのS1が参照
4. **パージ対象フラグリスト** — 定期クリーンアップバッチが参照

---

## Completion Message

R3 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — R3: Audit Log Finalization 完了

監査ログ:
- 記録エントリ数: [X]件
- 完全性チェック: [PASS / 差分あり（要確認）]
- 保存先: logs/audit/{YYYY-MM}/audit-{date}.log

サイクルサマリー:
- サイクルID: [cycle_id]
- 実行時間: [X]分
- ループ数: [N] / 5
- 終了理由: [all_completed / loop_limit / timeout]

次サイクル繰り延べ: [X]件
パージ対象フラグ: [X]件

サイクル状態: completed ✓

### WHAT'S NEXT
**A) 修正依頼** — 監査ログの内容や保持期間設定を修正する
**B) サイクル完了** — 次回巡回（約[X]分後）まで待機します
```

---

## Error Handling

### エラー 1: 監査ログ未記録によるアクション追跡不能（Pitfall 2対策）

**症状**: 監査ログを残さずにサイクルを完了した。誤った自動実行が発生しても原因追跡ができなかった。1週間後のレビューで「なぜこの商談のステージが変わったか不明」という問題が発生した。
**原因**: R3がエラーで停止した際に監査ログ書き込みが実施されなかった。
**対処**: 監査ログ書き込みは全ての他の処理より優先する。書き込み失敗時はリトライし、それでも失敗した場合はローカルバッファに保存して次サイクルで再送する。

**GOOD例**:
```
D1で record_activity を5件実行 + D3でsend_emailを1件承認実行
→ R3: 全6件のアクションを監査ログに記録
→ 翌週の承認却下率レビューで「send_email の却下率が60%」を発見
→ アクション判定ロジックの見直しフラグを発動 → 精度改善へ（Pitfall 2対策）
```

**BAD例**:
```
R3実行中に監査ログストレージへの接続エラー
→ エラーを無視してサイクルを完了
→ 監査ログなし
→ 1週間後: 「商談XのステージがなぜProposalに戻っているか不明」
→ 原因追跡ができず、同じ問題が繰り返し発生（Pitfall 2対策漏れ）
```

### エラー 2: 完全性チェック差分の放置

**症状**: 監査ログの記録アクション数（4件）と実際の実行数（5件）が一致しないが、完全性チェックをスキップして差分を放置する。1件のアクションが追跡不能になる。
**原因**: Step 3の完全性チェック実装が省略されている。
**対処**: 完全性チェックで差分が検出された場合は管理者に通知し、不整合の原因（どのアクションが記録されていないか）を特定して監査ログに追記する。

**GOOD例**:
```
完全性チェック:
→ D1実行数: 5件（completed）
→ 監査ログ記録数: 4件
→ 差分: 1件
→ 未記録アクションを特定: ACT-20260405-008（record_activity）
→ 事後記録を監査ログに追加 + 管理者通知: 「記録漏れ1件を検出・補完しました」
→ 完全性を回復
```

**BAD例**:
```
完全性チェック: 実行5件 vs 記録4件 → 差分1件（無視）
→ サイクルを完了
→ 1件のSalesforce Activity記録の証跡が消失
→ SOC 2監査時に「Activity記録の完全性を証明できない」と指摘
（コンプライアンスリスク）
```

### エラー 3: Salesforce内部への監査ログ保存によるAPI消費（Pitfall 4対策）

**症状**: 監査ログをSalesforce Custom Object に書き込んでいるため、監査ログ記録自体がSalesforce APIを消費し、ガバナーリミットをさらに圧迫する。
**原因**: 監査ログ保存先の設計誤り。
**対処**: 監査ログは必ずSalesforce外部（ファイルストレージ、外部ログサービス等）に保存する。Salesforce APIを消費しない設計を徹底する。

**GOOD例**:
```
監査ログ保存先: logs/audit/2026-04/audit-2026-04-05.log（ローカルファイル）
→ 書き込みAPI消費: 0（Salesforce APIを使用しない）
→ ガバナーリミットへの影響なし（Pitfall 4対策）
→ ログは外部ストレージにアップロード（バッチ処理で非同期）
```

**BAD例**:
```
監査ログ保存先: Salesforce Custom Object "Audit_Log__c"
→ 6件のアクション記録 → 6回のSalesforce API INSERT呼び出し
→ 監査ログ記録だけで1巡回あたり6 API消費
→ 日20サイクル × 6 = 120 API消費（ガバナーリミットを圧迫）（Pitfall 4対策漏れ）
```

### エラー 4: 繰り延べリストが次サイクルに引き継がれない

**症状**: R3でサイクル状態ファイルを `completed` に更新する際に繰り延べリストをクリアしてしまい、次サイクルのS1が繰り延べアクションを引き継げない。
**原因**: Step 5のサイクル完了更新で、繰り延べリストのフィールドを誤ってリセットしている。
**対処**: サイクル状態ファイルの `completed` 更新時は `cycle_status` と `completed_at` のみ更新する。繰り延べリスト（`deferred_actions_count`）や `suspended_action_types` は保持する。

**GOOD例**:
```
R3: cycle_status = "completed" に更新
→ 更新対象フィールド: cycle_status, completed_at のみ
→ deferred_actions_count: 2（保持）
→ suspended_action_types: []（保持）
→ 次サイクルのS1: deferred_actions_count = 2 を確認して繰り延べリストを引き継ぎ
→ 繰り延べアクションが次サイクルで正常処理
```

**BAD例**:
```
R3: サイクル状態ファイルを完全リセット（全フィールドをデフォルト値に）
→ deferred_actions_count = 0（誤リセット）
→ 次サイクルのS1: 繰り延べリストなし → 空の状態で巡回開始
→ D4でループ上限到達した2件の商談が次サイクルでも未対応
→ 繰り延べが連鎖して商談フォローアップが永続的に後回しに
```
