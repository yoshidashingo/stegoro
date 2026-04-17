# Red Team Review — REFINEMENT Phase

## Purpose
生成されたステアリングポリシーセット全体をPACKAGING前に敵対的視点でレビューする。2つの独立したエージェントが異なる観点から評価し、内部一貫性チェックや網羅性チェックでは検出できない致命的欠陥・論理的矛盾・品質問題を発見する。

## Prerequisites
- REFINEMENTフェーズ完了: Completeness Review + Consistency Review + Quality Calibration が PASS（またはCONDITIONAL APPROVAL）
- 全生成ポリシーファイルがディスクに書き込み済み
- `common/quality-standards.md` 読込済み

## Execution Classification
**Type**: ALWAYS — Quality Calibration完了後、必ず実行する

---

## Execution Steps

### Step 1: 並行レビューエージェント起動

**2つのエージェントを並行起動する**:

1. **`oh-my-claudecode:critic`** — 厳格なRed Teamレビュー
   - スコープ: `steering-docs/<agent-name>/` 配下の全生成ポリシーファイル
   - 役割: REJECT/ACCEPT判定、CRITICAL/MAJOR/MINOR finding分類、具体的改善提案

2. **`codex:codex-rescue`** — 定量分析・代替実装提案
   - スコープ: `steering-docs/<agent-name>/` 配下の全生成ポリシーファイル
   - 役割: ファイル別品質スコア、具体的な改善コード例

**両エージェントへのプロンプト**（ターゲットエージェントに合わせて調整）:

```
steering-docs/<agent-name>/ にある <ターゲットエージェントの説明> 向けステアリングポリシーセットをレビューしてください。

確認観点:
1. 生成されたルールはAIエージェントが実行可能な形で論理的に一貫しているか
2. ドメイン固有ルールは実際にドメイン固有か（汎用テンプレートのままでないか）
3. 安全ガード（allowlist、dry-run、承認ゲート）が正しく定義されているか
4. core-workflow.md とフェーズルールファイルの間に矛盾がないか
5. 品質基準（15次元）が定義通り計測可能か

報告: ACCEPT または REJECT、CRITICAL/MAJOR/MINOR findingをファイル:行番号付きで

REJECT基準: CRITICAL 1件以上
CONDITIONAL ACCEPT基準: CRITICAL 0件、MAJOR 3件以上
ACCEPT基準: CRITICAL 0件、MAJOR 2件以下
```

### Step 2: Finding収集・統合

両エージェントの完了を待ち、以下を実行する:
1. `oh-my-claudecode:critic` の findings を収集
2. `codex:codex-rescue` の findings を収集
3. 重複する指摘を統合・排除
4. 統合後 findings を重大度で分類

| 重大度 | 基準 | 例 |
|--------|------|----|
| **CRITICAL** | ポリシーが使用不能・矛盾・安全でない | core-workflow の Safety Check が存在しない |
| **MAJOR** | エージェント性能を著しく低下させる品質問題 | ドメイン固有ルールが汎用テンプレートのまま |
| **MINOR** | 改善推奨。リリースはブロックしない | 例示の粒度が不統一 |

### Step 3: 採否判定

| 判定 | 基準 | 次のアクション |
|------|------|---------------|
| **ACCEPT** | CRITICAL = 0, MAJOR ≤ 2 | PACKAGINGへ進む |
| **CONDITIONAL ACCEPT** | CRITICAL = 0, MAJOR ≥ 3 | 指摘を反映後、再レビュー不要でPACKAGINGへ |
| **REJECT** | CRITICAL ≥ 1 | 指摘を反映・修正後、Completeness Review (R1) から再実施 |

### Step 4: 0件指摘の方法論証跡

両エージェントが合計0件の場合、**方法論証跡が必須**:

- どのファイルを確認したか
- どの観点（安全性、ドメイン固有性、矛盾等）を確認したか
- なぜ問題がないと判断したか

**GOOD**: 「23ファイルをレビュー。安全性: core-workflowにallowlist・Safety Check定義済み。ドメイン固有性: 8/8フェーズルールファイルにドメイン固有用語あり。矛盾: core-workflowとフェーズルール間でステージ名一致確認。指摘0件。」
**BAD**: 「レビュー完了。問題なし。」 → 証跡なし → 再レビュー要求

### Step 5: Findings反映

REJECT または CONDITIONAL ACCEPT の場合:
1. 全 Critical/Major findings をファイル:行番号付きで列挙
2. 各 finding に対して: 問題の説明・必要な修正・修正後の内容を記述
3. 修正を対象ファイルに適用
4. 修正内容を audit.md に記録

### Step 6: ユーザー提示 (CP-RED)

```markdown
### CP-RED: Red Teamレビュー結果

**実行エージェント**: oh-my-claudecode:critic + codex:codex-rescue（並行実行）
**判定**: [ACCEPT / CONDITIONAL ACCEPT / REJECT]
**指摘件数**: [N]件 (Critical: [N], Major: [N], Minor: [N])

#### Critical指摘
[リスト または「なし」]

#### Major指摘
[リスト または「なし」]

#### Minor指摘
[リスト または「なし（N件）」]

---

**A) REJECTの場合** — CRITICAL指摘を反映後、Completeness Review (R1) から再実施します
**B) ACCEPT / CONDITIONAL ACCEPTの場合** — PACKAGINGフェーズへ進みます
```

---

## Output Artifacts

- Red Team Review Report: `steering-docs/<agent-name>/refinement/red-team-review-report.md`

---

## Error Handling

### エージェント利用不可
- **問題**: `oh-my-claudecode:critic` または `codex:codex-rescue` が利用不可
- **対処**: Step 1 の5観点をチェックリストとして手動レビューを実施。audit.md に「Manual Red Team Review」として記録
- **結果**: 最低 CONDITIONAL ACCEPT 扱い、ユーザーへ通知

### 両エージェントの判定が矛盾
- **問題**: critic が REJECT、codex が ACCEPT と判定
- **対処**: REJECT が優先。拒否エージェントの全 Critical/Major findings を反映してから再判定

## Pitfall References
- **H2O参照混入**: Source Contamination Scan (consistency-review.md Step 1b) を通過していても、文脈的な汚染（ドメイン特化のつもりがソースプロジェクトの業務手順を反映している等）はRed Teamで検出する
- **形式的レビュー**: 0件指摘は証跡なしに許容しない（Step 4参照）

## References
- `common/quality-standards.md` — 15品質次元
- `refinement/quality-calibration.md` — 前段階のキャリブレーション結果
- `refinement/consistency-review.md` — Source Contamination Scan結果
- `CLAUDE.md`（このリポ）— Red Teamレビュープロトコル
