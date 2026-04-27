# A3: Priority Ranking — Phase Rule

## Purpose

A2で生成したアクションリストに優先度スコアを付与し、実行順序を確定する。商談金額・クロージング期限・ヘルススコアの3軸による優先度スコアリングを適用し、P1〜P4の4段階に分類する。サイクル残り時間が15分以下の場合はP1・P2アクションのみ処理するタイムアウトリスク制御も実施する。

**Classification**: ALWAYS EXECUTE（日中モード・対応要案件あり時に必須）
**Approval Gate**: なし（次ステージ A4 へ自動移行）
**参照元**: `workflow-architecture.md` A3 セクション

---

## Prerequisites

- A2のアクションリスト（`state/action-list.yaml`）が確定していること
- A1のヘルススコアリスト（`state/health-scores.yaml`）が参照可能であること
- S1のサイクル状態ファイルで経過時間が把握できること
- 商談金額の大型定義閾値（`config/priority-config.yaml`: デフォルト ¥10,000,000）が設定済みであること

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: A2のアクションリストに1件以上のアクションが存在する場合
- Skip IF: アクション件数0件（このステージには到達しない）

---

## Execution Steps

### Step 1: 優先度スコアの算出

各アクションに対して以下の式で PriorityScore を算出する。

```
PriorityScore = (DealAmount / 1,000,000) × AmountWeight
              + (1 / max(DaysToClose, 1)) × DeadlineWeight × 100
              + (1 - HealthScore / 100) × UrgencyWeight × 100

AmountWeight  = 0.3
DeadlineWeight = 0.4
UrgencyWeight  = 0.3
```

**各パラメータの取得元**:

| パラメータ | 取得元 |
|----------|--------|
| `DealAmount` | S3統合ビュー `amount`（円） |
| `DaysToClose` | S3統合ビュー `close_date` と本日の差分（営業日） |
| `HealthScore` | A1ヘルススコアリスト `health_score`（0-100） |

**大型商談（¥10M以上）の補正**:
- `DealAmount >= 10,000,000` の場合: `AmountWeight` を 0.3 × 1.5 = **0.45** として計算
- 大型商談フラグ（`large_deal_flag: true`）が付与されている場合に自動適用

**DaysToClose = 0（当日クロージング）の処理**:
- `max(DaysToClose, 1)` により ÷0 を防止
- 当日クロージングは最大の DeadlineWeight 寄与となる

### Step 2: 優先度レベルの分類

算出した PriorityScore に基づいて P1〜P4 を付与する。

| 優先度 | 条件 | 処理方針 |
|--------|------|---------|
| P1（Critical） | スコア上位10%、**または** クロージング日が1日以内 | 同一サイクル内で必ず処理（繰り延べ対象外） |
| P2（High） | スコア上位30%（P1除く）、**または** クロージング日が7日以内 | 優先処理。タイムアウトリスク時まで処理 |
| P3（Medium） | スコア上位60%（P1・P2除く） | 通常処理。タイムアウトリスク時は次サイクルへ繰り延べ |
| P4（Low） | それ以外 | 余裕がある場合のみ処理。タイムアウトリスク時は繰り延べ |

**スコア上位X%の算出方法**:
- 全アクションのPriorityScoreを降順ソート
- 上位10%: `ceil(total_count × 0.10)` 件までをP1候補
- P1判定は「スコア上位10% **または** クロージング1日以内」のOR条件
- P2判定は「スコア上位30% **または** クロージング7日以内」のOR条件（P1除く）

**P1の繰り延べ禁止**:
- D4のループ上限到達（5ループ）またはタイムアウト（45分）時も、P1アクションは次サイクルへ繰り延べせず、現サイクル内で強制処理する
- ただし全ソース障害（S2で全接続失敗）の場合のみ例外的に繰り延べを許可

### Step 3: タイムアウトリスク制御

サイクル経過時間を確認し、残り時間が少ない場合はP3・P4アクションを繰り延べる。

**タイムアウトリスク判定**:
```
elapsed_time = current_time - cycle_start_time
remaining_time = 45分 - elapsed_time

IF remaining_time <= 15分:
  → タイムアウトリスクモード: P1・P2 のみ処理
  → P3・P4 アクションを次サイクル繰り延べリストに移動
  → サイクル状態ファイルに「timeout_risk: true」を記録
```

**タイムアウトリスクモード時の処理**:
- P3・P4アクションを `state/deferred-actions.yaml` に追記
- 繰り延べ理由: `deferred_reason: "timeout_risk_p1_p2_only"`
- 担当者への通知は不要（次サイクルで自動再処理）

### Step 4: 優先度付きアクションリストの確定

PriorityScore 降順でアクションリストを並び替え、各アクションに優先度情報を付与する。

```yaml
action_list_prioritized:
  generated_at: "2026-04-05T10:07:00+09:00"
  cycle_id: "cycle-uuid-xxxx"
  timeout_risk: false
  deferred_count: 0

  actions:
    - action_id: "ACT-20260405-003"
      priority: P1
      priority_score: 87.5
      action_type: "send_reminder"
      target_opportunity_name: "株式会社H — Enterpriseプラン"
      deal_amount: 12000000
      days_to_close: 1
      health_score: 28
      health_color: "red"
      large_deal_flag: true
      authority: auto
      sort_order: 1

    - action_id: "ACT-20260405-001"
      priority: P2
      priority_score: 62.3
      action_type: "send_email"
      target_opportunity_name: "株式会社B — ライセンス更新"
      deal_amount: 4200000
      days_to_close: 25
      health_score: 42
      health_color: "yellow"
      large_deal_flag: false
      authority: approval_required
      sort_order: 2
```

---

## Output Artifacts

A3 が生成するアーティファクト:

1. **優先度付きアクションリスト**（`state/action-list-prioritized.yaml`） — A4・D1・D2で参照
2. **繰り延べアクションリスト更新**（タイムアウトリスク時） — `state/deferred-actions.yaml` に追記
3. **タイムアウトリスクフラグ** — サイクル状態ファイルに記録

---

## Completion Message

A3 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — A3: Priority Ranking 完了

優先度分類:
- P1（Critical）: [X]件 — 今サイクル必須処理
- P2（High）: [X]件
- P3（Medium）: [X]件
- P4（Low）: [X]件

タイムアウトリスク: [なし / あり（P3・P4 X件を次サイクルに繰り延べ）]

### WHAT'S NEXT
**A) 修正依頼** — 優先度スコアの重みや閾値を修正する
**B) [A4 Authority Classification]へ続行** — 自動実行/承認制の分類へ進む
```

---

## Error Handling

### エラー 1: FIFO順処理による重要商談の処理漏れ

**症状**: 優先度を考慮せずFIFO順（検出順）でアクションを処理し、サイクルタイムアウト直前に大型商談の期限接近アラートが処理されなかった。翌日まで担当者に通知が届かず、クロージング機会を逸失した。
**原因**: Step 4の優先度スコア降順ソートが実装されていない。
**対処**: アクションリストは必ず PriorityScore 降順でソートしてから A4・D1・D2 に渡す。FIFO順の処理は禁止。

**GOOD例**:
```
アクション生成順: ACT-001（P4）→ ACT-002（P1: 明日クロージング ¥8M）→ ACT-003（P3）
→ PriorityScore降順ソート: ACT-002（87点）→ ACT-003（45点）→ ACT-001（12点）
→ ACT-002（P1）を最初に処理
→ クロージング前日のアラートが確実に担当者に届く
```

**BAD例**:
```
アクション生成順: ACT-001（P4）→ ACT-002（P1: 明日クロージング ¥8M）→ ACT-003（P3）
→ FIFO順で処理: ACT-001 → ACT-002 → ACT-003
→ タイムアウト直前（残り3分）に ACT-002 の処理開始
→ 承認リクエスト送信中にタイムアウト
→ クロージング前日なのに担当者に通知が届かない
```

### エラー 2: P1アクションの誤った繰り延べ

**症状**: ループ上限（5回）到達時にP1アクションも一括して次サイクルに繰り延べてしまい、当日クロージングの商談への緊急対応が翌サイクルまで遅延する。
**原因**: D4のループ制御が優先度を考慮せず全残件を繰り延べる実装になっている。
**対処**: A3でP1アクションには `no_defer: true` フラグを付与し、D4での繰り延べ処理でこのフラグを確認する。

**GOOD例**:
```
ループ5回到達: 残件 P1×1件 + P3×3件
→ P1アクション（no_defer: true）: ループ上限を超えても現サイクルで強制処理
→ P3アクション（3件）: 次サイクルへ繰り延べ
→ 当日クロージング商談の対応が今サイクルで確実に実施
```

**BAD例**:
```
ループ5回到達: 残件 P1×1件 + P3×3件
→ 全件を次サイクルへ繰り延べ
→ 当日クロージング商談へのアラートが翌サイクル（1時間後）まで遅延
→ 担当者が対応できずクロージングを逸失
```

### エラー 3: 残り時間未確認によるタイムアウトオーバーラン（Pitfall 4対策）

**症状**: Step 3のタイムアウトリスクチェックを実施せず、残り2分でP3アクション（10件）の処理を開始し、45分タイムアウトでサイクルが強制終了。P1アクションが未処理のまま残る。Salesforce APIの呼び出しが中断され、部分的な更新状態が残る（Pitfall 4: API制限への対応不足と同様の構造的リスク）。
**原因**: サイクル経過時間をA3で確認していない。
**対処**: A3の冒頭（Step 3）でサイクル経過時間を確認する。15分以下であればP3・P4を即座に繰り延べリストに移動してからA4へ進む。タイムアウトによる中断はAPI呼び出しの中途停止を引き起こし、Pitfall 4（API制限・中断リスク）と同様の影響を生む。

**GOOD例**:
```
サイクル経過時間: 32分（残り13分 < 15分閾値）
→ タイムアウトリスクモード発動
→ P3・P4（6件）を next-cycle deferred リストへ移動
→ P1（1件）・P2（2件）のみA4へ渡す
→ 残り13分でP1・P2（3件）を確実に処理
→ Salesforce API呼び出しが中途停止することなく完了（Pitfall 4対策）
```

**BAD例**:
```
サイクル経過時間確認なし
→ 全12件のアクションをA4へ渡す
→ D1でP4アクション（10件）のActivity記録を順次実行中に45分タイムアウト
→ P1アクション（1件）が未処理のまま強制終了
→ Salesforce Bulk APIジョブが中途停止し、一部レコードのみ更新された不整合状態に
（Pitfall 4対策漏れ: API呼び出しの中途停止リスク）
```
