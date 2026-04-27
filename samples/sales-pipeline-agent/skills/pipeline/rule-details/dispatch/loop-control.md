# D4: Loop Control — Phase Rule

## Purpose

残件チェック、ループカウンタ管理、承認タイムアウト処理、次ループまたは終了の判定を行う。最大5ループ・45分タイムアウトによる無限ループ防止、30分リマインダー・2時間繰り延べによる承認待ちブロック防止、ループ中の新規検出繰り延べによる通知爆発防止（Pitfall 3対策）を実施する。

**Classification**: ALWAYS EXECUTE（DISPATCH フェーズで必須）
**Approval Gate**: なし（D5または A1 へ自動移行）
**参照元**: `workflow-architecture.md` D4 セクション

---

## Prerequisites

- D1の実行完了記録が参照可能であること（または D1 がスキップされていること）
- D3の承認処理記録が参照可能であること（または D3 がスキップされていること）
- A4のアクションリストに残件カウントが記録されていること
- D2のタイムアウトスケジュール（`state/approval-requests.yaml`）が参照可能であること
- サイクル状態ファイルの `Loop Count` と `Start Time` が記録されていること

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: 常に実行（D1・D2・D3の後、またはA4でアクションなしの場合）
- Skip IF: なし（スキップ不可）

---

## Execution Steps

### Step 1: 残件カウントの算出

現在の未処理アクション数を確認する。

```
remaining_count = count(actions WHERE status NOT IN ['completed', 'rejected', 'deferred', 'rejected_this_cycle'])

内訳:
  auto_pending   = count(auto_execute_list WHERE status == 'pending')
  approval_pending = count(approval_requests WHERE status == 'pending')
  lock_deferred  = count(actions WHERE status == 'lock_timeout')
```

**残件0の判定条件**:
- `auto_pending == 0` かつ `approval_pending == 0`
- ただし「承認待ち（pending）」は残件としてカウントするため、承認リクエストを送信済みで返答待ちの場合は残件ありとなる

### Step 2: ループ継続・終了の判定

残件数・ループカウント・経過時間の3条件で判定する。

**判定ロジック**:
```
elapsed_time = current_time - cycle_start_time
loop_count = state.loop_count

# 終了条件（優先順に評価）
IF remaining_count == 0:
  → EXIT: 全件処理完了 → D5 へ

IF elapsed_time >= 45分:
  → TIMEOUT: 残件を次サイクルに繰り延べ → D5 へ
  → timeout_flag = true

IF loop_count >= 5:
  → LOOP_LIMIT: 残件を次サイクルに繰り延べ → D5 へ
  → loop_limit_flag = true

# P1アクションの強制処理チェック（上記終了条件より優先）
IF remaining_count > 0 AND exists(remaining_actions WHERE priority == 'P1' AND no_defer == true):
  → P1強制処理: ループカウントに関わらず A1 へ戻る（ただし elapsed_time >= 45分 は例外）
  → NOTE: タイムアウトの場合のみP1も繰り延べを許容

# 継続条件
IF remaining_count > 0 AND loop_count < 5 AND elapsed_time < 45分:
  → CONTINUE: loop_count += 1 → A1（Deal Health Scoring）へ戻る
```

**ループカウントの更新**:
```yaml
# サイクル状態ファイルへの記録
- Loop Count: [N] / Max: 5
- Elapsed: [X]分 / Timeout: 45分
```

### Step 3: 承認タイムアウト管理

D2で登録したタイムアウトスケジュールを確認し、30分リマインダーと2時間繰り延べを処理する。

**30分リマインダーの送信**:
```
FOR each request IN approval_requests WHERE status == 'pending':
  IF (current_time - request.sent_at) >= 30分
    AND request.reminder_sent_at IS NULL:

    # リマインダー送信
    slack.postMessage(
      channel = request.recipient_slack_id,
      text = "⏰ リマインダー: 承認リクエスト {request_id} が30分以上未対応です。\n" +
             "ご対応をお願いします。あと {2h - elapsed} で自動繰り延べになります。"
    )
    request.reminder_sent_at = current_time
```

**2時間繰り延べの実行**:
```
FOR each request IN approval_requests WHERE status == 'pending':
  IF (current_time - request.sent_at) >= 2時間:

    # 繰り延べ処理
    FOR each action IN request.actions WHERE result == 'pending':
      move_to_deferred_list(action, reason="approval_not_received_2h")

    request.status = "timed_out"
    slack.postMessage(
      channel = request.recipient_slack_id,
      text = "⌛ 承認リクエスト {request_id} が2時間以上未対応のため、次サイクルに繰り延べました。\n" +
             "次回巡回時（約1時間後）に再度リクエストを送信します。"
    )
```

**繰り延べアクションの記録**:
```yaml
deferred_actions:
  - action_id: "ACT-20260405-005"
    deferred_at: "2026-04-05T12:10:00+09:00"
    deferred_reason: "approval_not_received_2h"
    original_request_id: "APPR-20260405-002"
    next_cycle_priority: P2  # 元の優先度を維持
```

### Step 4: ループ中の新規検出繰り延べ確認（Pitfall 3対策）

ループ中（loop_count >= 1）に新たに検出された案件が現サイクルのアクションリストに追加されていないかを確認する。

**確認内容**:
- A1〜A4の再実行時に新規生成されたアクションが `action_list` に追加されていた場合、それらを即座に次サイクルの繰り延べリストへ移動する

```
IF loop_count >= 1:
  new_actions = filter(action_list, created_at > cycle_start_time + first_loop_duration)
  FOR each new_action IN new_actions:
    IF new_action.opportunity_id NOT IN original_detection_opportunity_ids:
      move_to_next_cycle_queue(new_action)
      log("新規検出を次サイクルへ繰り延べ: {new_action.target_opportunity_name}")
```

### Step 5: 繰り延べ残件の記録（終了時のみ）

ループ終了（残件0 / ループ上限 / タイムアウト）の場合、残件を繰り延べリストに記録する。

```yaml
# state/deferred-actions.yaml への追記
- action_id: "ACT-20260405-009"
  deferred_at: "2026-04-05T10:45:00+09:00"
  deferred_reason: "loop_limit_5_reached"  # または "cycle_timeout_45min"
  priority: P3
  target_opportunity_name: "株式会社J — ライセンス更新"
```

**繰り延べ件数をサイクル状態ファイルに記録**:
```markdown
- [timestamp] D4: Loop [N] complete → [exit_reason]
  Remaining deferred: [X]件
```

---

## Output Artifacts

D4 が生成するアーティファクト:

1. **ループ判定結果**（継続 → A1 / 終了 → D5） — フロー制御
2. **繰り延べアクションリスト更新**（`state/deferred-actions.yaml`）
3. **承認リマインダー送信記録** — 監査ログに追記
4. **タイムアウト / ループ上限フラグ** — R1のサマリーに記録

---

## Completion Message

D4 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — D4: Loop Control 完了

ループ状態:
- 現在のループ数: [N] / 5
- 経過時間: [X]分 / 45分
- 残件数: [X]件

判定結果:
- [継続（A1へ戻る）/ 完了（残件0）/ ループ上限到達 / タイムアウト]

承認タイムアウト処理:
- 30分リマインダー送信: [X]件
- 2時間繰り延べ: [X]件

次サイクル繰り延べ（合計）: [X]件

### WHAT'S NEXT
**A) 修正依頼** — ループ制御の閾値や繰り延べ条件を修正する
**B) [A1 Deal Health Scoring]へ戻る** — 継続時（ループN+1回目）
**C) [D5 Rollback Check]へ続行** — 終了時
```

---

## Error Handling

### エラー 1: 無限ループによる次巡回スケジュール超過（Pitfall 2対策）

**症状**: 承認待ちアクションが解決するまで無限にループし、次の巡回タイミング（1時間後）を超過してスケジュールが乱れる。翌サイクルが二重起動される。
**原因**: Step 2のループ上限チェック（loop_count >= 5）またはタイムアウトチェック（elapsed >= 45分）が実装されていない。
**対処**: 必ず両方の終了条件をチェックする。どちらかに達した時点で即座にD5へ進む。

**GOOD例**:
```
ループカウント 5回到達: 残件 P3×2件
→ LOOP_LIMIT: 残件2件を次サイクルへ繰り延べ
→ D5へ進み、サイクルを正常完了
→ 次の1時間サイクルが正常に起動
→ 繰り延べ2件が次サイクルで処理される（Pitfall 2対策）
```

**BAD例**:
```
承認待ちアクション（P3×2件）が解決するまでループ継続
→ 2時間経過してもループ継続（ループ上限なし）
→ 45分後に予定していた次サイクルが二重起動
→ 2つのサイクルが同時に実行され、同一商談への重複更新が発生
（ループ制御なし）
```

### エラー 2: 承認待ちでエージェントがブロック（Pitfall 2対策）

**症状**: 承認リクエストへの返答を待つためにループを継続し続け、担当者が長期不在の場合にエージェントが45分間何もできない状態になる。
**原因**: 承認待ちアクションを「残件」としてカウントし、2時間繰り延べ処理を実装していない。
**対処**: 承認待ちアクションは2時間後に自動的に次サイクルへ繰り延べ、エージェントを承認待ちでブロックしない。

**GOOD例**:
```
承認リクエスト送信: 10:10
→ ループ継続: 残件（承認待ち）あり
→ 10:40（30分後）: リマインダー送信
→ 12:10（2時間後）: 承認待ちアクションを次サイクルへ繰り延べ
→ 残件0 → D5へ → サイクル完了
→ 担当者は非同期で承認可能（次サイクルでも対応される）
```

**BAD例**:
```
承認リクエスト送信: 10:10
→ 承認待ちが「残件」のままループ継続
→ 担当者が会議中で12:00まで未応答
→ エージェントが10:10〜10:55（45分タイムアウト）まで承認待ちでブロック
→ 他の商談への対応も遅延
（承認待ちブロック対策なし）
```

### エラー 3: ループ中の新規検出を現サイクルに追加（Pitfall 3対策）

**症状**: ループ2回目でA1〜A4を再実行した際に新たに検出された5件の商談が現サイクルのアクションリストに追加され、担当者が同日に大量の通知を受け取る。
**原因**: Step 4の新規検出繰り延べチェックが実装されていない。
**対処**: ループ中（loop_count >= 1）に新規生成されたアクションは、元の検出セットにない商談については全て次サイクルの繰り延べキューに移動する。

**GOOD例**:
```
ループ1回目: 商談A・B・Cのアクションを処理
→ A1再評価時: 商談D・Eも新たに検出
→ D4: 商談D・Eは original_detection_opportunity_ids に未含 → 次サイクルキューへ移動
→ 現サイクルは商談A・B・Cのみ継続処理
→ 担当者への通知は商談A・B・Cのみ（Pitfall 3対策）
```

**BAD例**:
```
ループ1回目: 商談A・B・Cのアクションを処理
→ A1再評価時: 商談D・Eも検出 → アクションリストに追加
→ 同サイクルで商談A〜Eの全アクションを処理
→ 担当者が同日に A・B・C・D・Eの通知を受け取る（通知爆発）
（Pitfall 3対策漏れ）
```
