# A1: Deal Health Scoring — Phase Rule

## Purpose

S4の検出結果リストに含まれる各商談のヘルススコアを算出する。エンゲージメント・ステージ進行・期限リスクの3軸を複合評価し、赤（0-39）/ 黄（40-69）/ 緑（70-100）のカラー分類を付与する。Salesforce単独データへの依存を防ぐため、Gmail/Slack/Calendarのアクティビティによる多層補正（Pitfall 5対策）を必ず適用する。

**Classification**: ALWAYS EXECUTE（日中モード・対応要案件あり時に必須）
**Approval Gate**: なし（次ステージ A2 へ自動移行）
**参照元**: `workflow-architecture.md` A1 セクション

---

## Prerequisites

- S4の検出結果リスト（`state/detection-results.yaml`）が確定していること
- S3の統合ビュー（`state/integrated-view.yaml`）が参照可能であること
- ステージ別受注確率テーブル（`config/stage-win-rates.yaml`）が設定済みであること
- 前回サイクルのヘルススコア記録（`state/health-score-history.json`）が参照可能であること（トレンド算出用）

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: S4の検出結果リストに1件以上の案件が存在する場合
- Skip IF: S4で検出件数0件（このステージには到達しない）

---

## Execution Steps

### Step 1: スコア計算式の適用

各商談に対して以下の式でHealthScoreを算出する。

```
HealthScore = (EngagementScore × 0.4) + (StageProgressScore × 0.3) + (DeadlineScore × 0.3)
```

**EngagementScore（エンゲージメントスコア）の算出**:

| 指標 | 評価方法 | 最大配点 |
|------|---------|---------|
| 最終アクティビティ経過日数 | 0日=100点、14日=50点、30日=0点（線形補間） | 40点 |
| メール応答率（直近5通） | 応答あり/送信数 × 100 | 30点 |
| ミーティング頻度（直近30日） | 2回以上=30点、1回=15点、0回=0点 | 30点 |

```
EngagementScore = (latest_activity_score × 0.4)
                + (email_reply_rate × 0.3)
                + (meeting_frequency_score × 0.3)
```

**StageProgressScore（ステージ進行スコア）の算出**:

| 指標 | 評価方法 | 最大配点 |
|------|---------|---------|
| ステージ滞留日数と閾値の比率 | (1 - days_in_stage / threshold) × 100。上限100、下限0 | 60点 |
| MEDDIC/BANT充足率 | `config/meddic-fields.yaml` で定義した必須フィールドの入力率 | 40点 |

```
StageProgressScore = (stage_progress_ratio × 0.6)
                   + (meddic_bant_fulfillment_rate × 0.4)
```

**DeadlineScore（期限リスクスコア）の算出**:

| クロージング日まで | スコア |
|----------------|--------|
| 31日以上 | 100 |
| 15〜30日 | 75 |
| 8〜14日 | 50 |
| 3〜7日 | 25 |
| 2日以内 | 10 |
| 期限超過 | 0 |

```
DeadlineScore = days_to_close_score（上表参照）
```

### Step 2: 多層補正の適用（Pitfall 5対策）

Salesforce上の情報のみからスコアが低く算出された場合でも、他ソースに活発なアクティビティがあればEngagementScoreを補正する。

**補正条件と補正値**:

| 条件 | 補正内容 |
|------|---------|
| Gmail/Slackで直近7日以内にアクティビティあり かつ Salesforce LastActivityDateが7日超過 | EngagementScore の `latest_activity_score` を100点に修正 |
| Slackで直近3日以内に商談関連メッセージあり | EngagementScore に +10点のボーナス（上限100） |
| Google Calendarで直近14日以内にミーティング実施 かつ Salesforce Activityに記録なし | `meeting_frequency_score` を記録ありとして評価 |

**補正適用のログ記録**:
補正を適用した場合は補正前後のスコアと補正理由を監査ログに記録する。

```yaml
health_score_audit:
  opportunity_id: "006xxxxxxxxxxxx"
  pre_correction_score: 35
  post_correction_score: 62
  correction_applied: true
  correction_reason: "Slackで直近3日のアクティビティを検出（Salesforce未記録）"
```

### Step 3: カラー分類の付与

| スコア範囲 | カラー | 意味 |
|-----------|--------|------|
| 70-100 | 緑（Green） | 良好。特段の緊急対応不要 |
| 40-69 | 黄（Yellow） | 注意。フォローアップ推奨 |
| 0-39 | 赤（Red） | 要緊急対応。早急なアクションが必要 |

**大型商談（¥10M以上）の特別扱い**:
- 黄（40-69）であっても `large_deal_flag: true` を付与し、A3の優先度スコアリングでAmountWeightを1.5倍に設定

### Step 4: 加重パイプライン予測の算出

検出対象商談の加重パイプライン金額を算出し、R1のサマリーに含める。

**ステージ別受注確率テーブル**（`config/stage-win-rates.yaml` 参照）:

| ステージ | デフォルト受注確率 |
|---------|----------------|
| Prospecting | 10% |
| Qualification | 25% |
| Value Proposition | 35% |
| Id. Decision Makers | 40% |
| Perception Analysis | 45% |
| Needs Analysis | 50% |
| Proposal/Price Quote | 50% |
| Negotiation/Review | 75% |
| Closed Won | 100% |
| Closed Lost | 0% |

```
weighted_pipeline_contribution = amount × stage_win_rate
```

### Step 5: トレンド算出と記録

前回サイクルのヘルススコアと比較し、スコアの変化トレンドを算出する。

```yaml
health_score_result:
  opportunity_id: "006xxxxxxxxxxxx"
  opportunity_name: "株式会社B — ライセンス更新"
  health_score: 42
  color: "yellow"
  large_deal_flag: false
  engagement_score: 55
  stage_progress_score: 30
  deadline_score: 50
  correction_applied: false
  previous_score: 48
  trend: "declining"  # improving / stable / declining
  weighted_pipeline: 2100000  # ¥4.2M × 50%
```

---

## Output Artifacts

A1 が生成するアーティファクト:

1. **ヘルススコアリスト**（`state/health-scores.yaml`） — A2・A3で参照
2. **加重パイプライン合計** — R1のサマリーに記載
3. **スコア補正ログ** — 監査ログに追記
4. **ヘルススコア履歴の更新**（`state/health-score-history.json`） — 次サイクルのトレンド算出に使用

---

## Completion Message

A1 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — A1: Deal Health Scoring 完了

ヘルススコア集計:
- 赤（要緊急対応）: [X]件
- 黄（注意）: [X]件
- 緑（良好）: [X]件

多層補正適用: [X]件（Salesforceのみの評価から補正）
加重パイプライン合計: ¥[X]M

### WHAT'S NEXT
**A) 修正依頼** — スコア計算式やステージ別閾値を修正する
**B) [A2 Action Identification]へ続行** — 具体的な対応アクションの特定へ進む
```

---

## Error Handling

### エラー 1: Salesforce単独スコアリングによる誤判定（Pitfall 5対策）

**症状**: SalesforceのLastActivityDateのみでEngagementScoreを算出し、担当者がSlackで活発に交渉中の商談を「赤（要緊急対応）」と誤判定する。
**原因**: Step 2の多層補正処理が実装されていない。
**対処**: EngagementScore算出時は必ず統合ビューの `latest_activity_date`（全ソース横断）を使用し、Salesforce LastActivityDateは補助指標として扱う。

**GOOD例**:
```
商談C: Salesforce LastActivityDate = 2026-03-20（16日前）
→ Salesforceのみ: EngagementScore = 20（赤判定候補）
→ 多層補正確認: Slack #deal-updates に 2026-04-04（1日前）の言及あり
→ latest_activity_score を 100点に修正
→ 補正後 EngagementScore: 75 → HealthScore: 66（黄判定）
→ 誤った緊急対応アラートを回避（Pitfall 5対策）
```

**BAD例**:
```
商談C: Salesforce LastActivityDate = 2026-03-20（16日前）
→ Salesforceのみで評価 → HealthScore: 28（赤判定）
→ 担当者にP1緊急アラート「商談Cが危機的状況」送信
→ 担当者: 「毎日Slackで交渉中なのに何が危機なのか」と不信感（Pitfall 1・5対策漏れ）
```

### エラー 2: MEDDIC/BANTフィールド未設定によるスコア過小評価

**症状**: `config/meddic-fields.yaml` が設定されていないため全商談のMEDDIC/BANT充足率が0%となり、StageProgressScoreが一律に低くなる。
**原因**: 設定ファイルの初期化忘れ、またはフィールドマッピングの設定漏れ。
**対処**: `config/meddic-fields.yaml` が存在しない場合、MEDDIC/BANT充足率を50%（デフォルト中立値）として扱い、スコアへの影響を最小化する。設定不備を管理者に通知する。

**GOOD例**:
```
config/meddic-fields.yaml が未設定
→ MEDDIC/BANT充足率をデフォルト 50% として適用
→ StageProgressScore への過小影響を防止
→ 管理者通知: 「MEDDIC/BANTフィールド設定が未完了です。精度向上のため config/meddic-fields.yaml を設定してください」
```

**BAD例**:
```
config/meddic-fields.yaml が未設定 → MEDDIC/BANT充足率 = 0%
→ 全商談の StageProgressScore が最大 60点まで低下
→ 健全な商談も黄・赤に誤判定
→ 不要なフォローアップアクションが大量生成（Pitfall 3の間接的原因）
```

### エラー 3: トレンド算出のための履歴スコア不在

**症状**: 初回実行または履歴ファイル消失により、前回スコアが参照できず `trend` が算出できない。
**原因**: `state/health-score-history.json` が存在しない状態でトレンド算出を試みる。
**対処**: 履歴がない場合は `trend: "unknown"` として記録し、スコア計算自体はスキップしない。次サイクル以降から正常なトレンド算出を開始する。

**GOOD例**:
```
state/health-score-history.json が存在しない（初回実行）
→ HealthScore を正常算出
→ trend: "unknown"（履歴なし）
→ 現在のスコアを history に新規記録
→ 次サイクル以降はトレンドが算出可能になる
```

**BAD例**:
```
state/health-score-history.json が存在しない
→ トレンド算出で NullPointerError
→ A1 全体がエラーで停止
→ S4で検出した全案件の対応が今サイクルで実施されない
```
