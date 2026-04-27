# P2: Automated Validation（自動バリデーション）

## Purpose

P1で生成したプラグイン構造に対して3層テスト（構造テスト・コンテンツテスト・スモークテスト）を実行し、PASS/FAIL基準に基づいて合否を判定する。FAILの場合はP1に戻るループ制御を行う（最大2回）。全9フローのスモークテストを完走してPASSした場合のみ、パッケージングを完了とする。

**Classification**: ALWAYS EXECUTE

**参照元**: `core-workflow.md` P2（BUILD-TIME: PACKAGING）

---

## Prerequisites

- P1（Plugin Structure Generation）が完了し、プラグイン構造が生成されていること
- 29ファイル（core 1 + common 12 + phase rules 16）が存在すること
- P2→P1ループカウンターが参照可能であること（最大2回）

---

## Execution Steps

### Step 1: Layer 1 — 構造テスト

ファイル存在・相互参照・Markdown妥当性・フローパスの検証を行う。

```
構造テストチェックリスト:

□ ファイル存在確認（29ファイル）
  対象: core-workflow.md(1) + common/(12) + triage/(3) + response/(5)
        + quality/(4) + follow-up/(2) + packaging/(2)
  PASS基準: 全29ファイルが存在する
  FAIL: 1ファイルでも欠損

□ 相互参照解決
  対象: core-workflow.md内の全 `rule-details/` 参照
  PASS基準: 参照先ファイルが100%存在する
  FAIL: 1件でも参照切れ

□ Markdown妥当性
  対象: 全29ファイル
  チェック項目: 見出し階層の飛びなし（## → ### は OK、## → #### は NG）
               コードブロックの開閉一致
               リスト構造の正常性
  PASS基準: 全ファイルでエラーなし

□ フローパス完全性
  対象: core-workflow.md のフロー定義
  チェック項目: デッドエンド（どこにも続かないステージ）が存在しないこと
               全CONDITIONAL分岐にSkip IFが定義されていること
  PASS基準: デッドエンドゼロ

□ ファイル数整合性
  設計書（phase-rules-design.md）との一致を確認
  PASS基準: 設計書の16ファイル + common 12 + core 1 = 29に一致
```

**Layer 1結果**: 全チェックPASS → Layer 2へ。1件でもFAIL → FAILレポート生成してP1へ。

### Step 2: Layer 2 — コンテンツテスト

品質次元（Dim 12〜15）の閾値達成を検証する。

```
コンテンツテストチェックリスト:

□ Dim 12: ドメイン特化率（>= 40%/ファイル）
  計算式: ドメイン固有行数 ÷ (総行数 - 空行 - 見出し行)
  ドメイン固有行の定義:
    - SaaS/サポート用語（チケット・KB・SLA・CSAT・エスカレーション等）
    - 正規表現パターン
    - テンプレート・JSONコード
    - 具体的な閾値・数値（例: KB信頼度0.6、90日、7日）
  対象: 全16 Phase Ruleファイル
  PASS基準: 全ファイルで >= 40%
  FAIL: 1ファイルでも < 40%

□ Dim 13: GOOD/BAD例カバレッジ（>= 2例/ファイル）
  対象: 全16 Phase Ruleファイル
  PASS基準: 全ファイルでGOOD例 >= 2件 かつ BAD例 >= 2件
  FAIL: 1ファイルでも不足

□ Dim 14: アーティファクトテンプレート完備率（= 100%）
  対象（6ファイル必須）:
    R3: 回答メールテンプレート
    R4: 引き継ぎサマリー + 中間応答テンプレート
    R5: 傾向レポートテンプレート
    Q4: 監査ログフォーマット
    F1: フォローアップ追跡記録フォーマット
    F2: KB改善提案テンプレート
  PASS基準: 6ファイル全てにテンプレートが存在する（6/6 = 100%）
  FAIL: 1ファイルでもテンプレート欠損

□ Dim 15: 4 Pitfalls引用率（>= 50%）
  4 Pitfalls:
    Pitfall 1: ハルシネーション回答
    Pitfall 2: PII漏洩
    Pitfall 3: エスカレーション判定ミス
    Pitfall 4: トーンミスマッチ
  計算: Error Handlingセクション内でPitfallsを引用するアイテム ÷ 全エラーアイテム
  対象: 全16 Phase Ruleファイルのエラーハンドリングセクション
  PASS基準: >= 50%
  FAIL: < 50%

□ フィードバックループカバレッジ
  対象: common/ + quality/ のルールファイル
  チェック: CSAT → F2 → KB改善の手順が記述されていること
  PASS基準: フィードバックループが追跡可能

□ 完了メッセージフォーマット
  対象: 全ステージ（core-workflow.md + 全Phase Ruleファイル）
  チェック: 「REVIEW REQUIRED」+ 2-option（A/B）形式が全ステージに存在すること
  PASS基準: 全ステージで2-option形式
```

**Layer 2結果**: 全チェックPASS → Layer 3へ。1件でもFAIL → FAILレポート生成してP1へ。

### Step 3: Layer 3 — スモークテスト（9フロー検証）

全5つの運用フローパターンと追加4フローが到達可能かを検証する。

```
スモークテストフロー一覧:

フロー1: 即時回答フロー
  T1 → T2 → T3 → R1 → R2 → R3 → Q1 → Q2 → Q3 → Q4
  PASS基準: 全ステージが定義済み。デッドエンドなし。

フロー2: エスカレーションフロー（条件ベース）
  T1 → T2 → T3 → R1(エスカレーション条件) → R4 → Q1 → Q2 → Q3 → Q4
  PASS基準: R1の条件ベースエスカレーション → R4 → QUALITYフェーズ通過が定義済み

フロー3: エスカレーションフロー（KB未ヒット）
  T1 → T2 → T3 → R1 → R2(KB信頼度<0.4) → R4 → Q1 → Q2 → Q3 → Q4
  PASS基準: KB信頼度<0.4でR4切り替えが定義済み

フロー4: エスカレーションフロー（Q3リペアパス）
  …→ Q3(即エスカレーション判定) → R4 → Q1 → Q2 → Q3 → Q4
  PASS基準: Q3リペアパスが定義済み（max 1回）。2回目は強制エスカレーション。

フロー5: 分析フロー
  T1 → T2 → T3 → R1(分析モード) → R5 → Q1 → Q2 → Q3 → Q4
  PASS基準: R5 → QUALITYフェーズ通過が定義済み

フロー6: フォローアップフロー
  Q4 → F1（条件付き） → F2（条件付き）
  PASS基準: F1のExecute IF条件が定義済み。F2のトリガー条件が定義済み。

フロー7: SLA超過パス
  T3(Warning) → … → Q1(必須) → Q2(簡易) → Q3 → Q4(超過フラグ)
  PASS基準: Q1のSLA問わず必須が定義済み。Q2の簡易チェックモードが定義済み。

フロー8: PACKAGINGフロー
  P1 → P2
  PASS基準: P1→P2の順序と P2→P1ループ（max2回）が定義済み

フロー9: Hybrid切り替え（5パターン全分岐）
  R1の3モード + R2のKB信頼度分岐 + Q3リペアパス = 計5パターン
  PASS基準: 全5パターンで条件が明確かつ重複なし
```

**Layer 3結果**: 全9フローPASS → バリデーション完了（PASS）。1フローでもFAIL → FAILレポート生成してP1へ。

### Step 4: 総合判定とループ制御

```
判定ロジック:
  Layer 1 PASS AND Layer 2 PASS AND Layer 3 PASS:
    → 総合判定: PASS
    → バリデーションレポートを生成してユーザーへ提示

  いずれかのレイヤーがFAIL:
    → ループカウンターを確認
      カウンター < 2:
        → カウンターをインクリメント
        → FAILレポートをP1に渡してP1を再実行
      カウンター >= 2:
        → P2_P1_LOOP_LIMIT_REACHED
        → ユーザーにエスカレーション: 自動修復上限到達
```

### Step 5: バリデーションレポート生成

```
[P2 VALIDATION REPORT]
- Timestamp: {ISO 8601}
- Loop Count: {N}/2

Layer 1 — 構造テスト: {PASS | FAIL}
  - ファイル存在: {29/29 | 不足: N件}
  - 相互参照: {100% | 未解決: N件}
  - Markdown妥当性: {PASS | FAIL: N件のエラー}
  - フローパス: {PASS | デッドエンド: N件}

Layer 2 — コンテンツテスト: {PASS | FAIL}
  - Dim 12（ドメイン特化率）: {全ファイルPASS | FAIL: N件 < 40%}
  - Dim 13（GOOD/BAD例）: {全ファイルPASS | FAIL: N件不足}
  - Dim 14（テンプレート）: {6/6 | N/6}
  - Dim 15（Pitfall引用率）: {N% | >= 50%でPASS}

Layer 3 — スモークテスト: {PASS | FAIL}
  - フロー1（即時回答）: {PASS | FAIL}
  - フロー2（エスカレーション条件）: {PASS | FAIL}
  - フロー3（エスカレーションKB）: {PASS | FAIL}
  - フロー4（Q3リペアパス）: {PASS | FAIL}
  - フロー5（分析）: {PASS | FAIL}
  - フロー6（フォローアップ）: {PASS | FAIL}
  - フロー7（SLA超過）: {PASS | FAIL}
  - フロー8（PACKAGING）: {PASS | FAIL}
  - フロー9（Hybrid5パターン）: {PASS | FAIL}

総合判定: {PASS | FAIL（P1へ: {N}回目）| LOOP_LIMIT_REACHED}
```

---

## Question Generation

バリデーション完了後、以下を提示する。

```
### バリデーション結果
総合判定: {PASS / FAIL}

{PASSの場合}:
全3層のテストが通過しました。プラグインのリリース準備が整いました。

{FAILの場合}:
以下の項目がFAILしました:
- {FAIL項目の詳細}
P1に戻り修正を実施します（{N}回目/2回上限）。

A) 修正依頼 — FAILの対応方法を変更したい場合、内容を指定してください
B) P1修正→P2再実行へ続行 — 自動修正後に再バリデーションを実行する
```

---

## Output Artifacts

- **バリデーションレポート**: 3層テスト全結果
- **FAILレポート**（FAILの場合）: P1への修正指示
- **ループカウンター更新**: P2→P1ループの回数管理

---

## Completion Message

```
### REVIEW REQUIRED
[バリデーション完了]
- 総合判定: {PASS / FAIL}
- Layer 1（構造）: {PASS / FAIL}
- Layer 2（コンテンツ）: {PASS / FAIL}
- Layer 3（スモーク 9フロー）: {PASS / FAIL}
- ループ回数: {N}/2

### WHAT'S NEXT
**A) 修正依頼** — テスト結果に対して追加確認・修正が必要な場合、内容を指定してください
**B) パッケージング完了** — PASS確認済み。プラグインをリリース可能状態にする
```

---

## Error Handling

### エラー1: Dim 12（ドメイン特化率）が閾値未満

**シナリオ**: 特定のPhase RuleファイルのドメインSpecificity率が35%でFAIL

**対応**:
1. 対象ファイル名と現在の特化率をFAILレポートに明示
2. P1に差し戻し（ファイル単体の再生成を指示）
3. 「汎用的なルール記述をサポートドメイン固有の表現に置き換えること」を修正指示に含める

### エラー2: スモークテストのフロー4（Q3リペアパス）がFAIL

**シナリオ**: Q3リペアパスの定義が不完全（max 1回の制限が記述されていない）

**原因候補**:
- Pitfall 3（エスカレーション判定ミス）派生: ループ制御の欠如がエスカレーション漏れを引き起こす

**対応**:
1. `quality/tone-calibration.md` の Step 4（リペアパス判定）を確認
2. リペアカウンター制御（max 1回、2回目は強制エスカレーション）の記述を修正してP1再実行

### エラー3: P2→P1ループが2回に到達しても FAIL が継続

**対応**:
1. `P2_P1_LOOP_LIMIT_REACHED` を記録
2. FAILした全項目をまとめて設計エラーレポートを生成
3. 「design-v2/ の設計書との乖離が疑われます。Workflow Architecture または Quality Mechanisms の再設計をご検討ください」を通知
4. 手動レビューを促す

---

## GOOD / BAD 具体例

### GOOD 例1: 3層テスト全PASS

**シナリオ**: 29ファイル全揃い。Dim 12は最低ファイルで42%。全9フロー到達可能。

**GOOD（バリデーション結果）**:
```
Layer 1: PASS（29/29ファイル、参照100%解決、Markdown正常、デッドエンドゼロ）
Layer 2: PASS（Dim12: 最低42%、Dim13: 全ファイル2例以上、Dim14: 6/6、Dim15: 62%）
Layer 3: PASS（9フロー全完走）
総合: PASS
```

### GOOD 例2: Layer 2 FAILでP1へ修正ループ（1回目）

**シナリオ**: Dim 13（GOOD/BAD例）でF1の `case-tracking.md` に BAD例が1件しかない。

**GOOD（ループ制御）**:
```
Layer 2 FAIL: Dim 13 — follow-up/case-tracking.md にBAD例が1件不足
ループカウンター: 1/2
→ P1に差し戻し: case-tracking.md の BAD例を2件以上に増補
→ P2を再実行
```

### BAD 例1: スモークテストをLayer 3のみで行い、Layer 1・2をスキップ

**BAD（NG行動）**: 「フローが動けばいい」としてLayer 1・2を省略し、Layer 3のみ実行。

**結果**: Markdown見出し階層の不整合（Layer 1 FAIL相当）やDim 14テンプレート欠損（Layer 2 FAIL相当）が見逃され、運用時に不具合が発生。

**GOOD**: 3層テストは順序通りに全て実行。Layer 1 FAIL時点でLayer 2・3は実行不要（FAILレポートをP1へ渡す）。

### BAD 例2: ループ制限を無視した3回目のP1再実行

**BAD（NG行動）**: P2→P1ループを2回実施してもFAILが続くため、3回目のP1再実行を試みる。

**GOOD（正しい処理）**: ループカウンターが2に達した時点で `P2_P1_LOOP_LIMIT_REACHED` を記録し、ユーザーにエスカレーション。設計レベルの問題として手動レビューを促す。自動ループの暴走を防止。
