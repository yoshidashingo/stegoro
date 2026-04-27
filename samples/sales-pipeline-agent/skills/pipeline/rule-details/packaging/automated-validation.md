# P2: Automated Validation — Phase Rule

## Purpose

P1で生成したプラグイン構造に対して3層テスト（構造テスト・コンテンツテスト・スモークテスト）を実行し、PASS/FAIL基準で品質を検証する。全層PASSで検証完了とし、FAILの場合はP1へ戻るリペアループを最大2回実行する。検証結果レポートを生成してパッケージングを完了する。

**Classification**: ALWAYS EXECUTE（ビルドタイム）
**Approval Gate**: あり（3層テスト全PASS）
**参照元**: `workflow-architecture.md` Build-Time: PACKAGING セクション

---

## Prerequisites

- P1のプラグイン構造生成が完了していること（CP-8 PASS）
- 全18ファイルのPhase Rulesが存在していること
- `plugin.json` が生成済みであること
- P1の整合性チェック結果が参照可能であること

---

## Execution Classification

**ALWAYS EXECUTE**（ビルドタイム）

- Execute IF: P1が正常完了している場合
- Skip IF: P1が失敗・停止している場合（P2は実行しない）

---

## Execution Steps

### Step 1: 第1層テスト — 構造テスト

プラグインの物理的な構造（ファイル・ディレクトリ・参照パス）の完全性を検証する。

**テスト項目**:

| テスト | 判定基準 | FAIL条件 |
|--------|---------|---------|
| 必須ファイル存在確認 | 18 Phase Rules + core-workflow.md + action-authority.md が全て存在 | 1ファイルでも欠損 |
| ディレクトリ構造 | surveillance/assessment/dispatch/reporting/packaging/ が存在 | ディレクトリ欠損 |
| plugin.json バリデーション | JSONとして有効かつ必須フィールド（name/version/entry_point）が存在 | JSONパースエラー または 必須フィールド欠損 |
| 参照パス整合性 | plugin.json 内の全参照先ファイルが存在 | 1参照先でも不在 |
| state/ logs/ の存在 | 実行時ディレクトリが存在（空でも可） | ディレクトリ不在 |

**チェックスクリプト相当の処理**:
```
structure_test_results = {
  "required_files": check_all_18_phase_rules_exist(),
  "directory_structure": check_directories_exist(),
  "plugin_json_valid": validate_plugin_json_schema(),
  "reference_integrity": check_all_references_exist(),
  "runtime_dirs": check_state_logs_exist()
}

layer1_pass = all(v == "PASS" for v in structure_test_results.values())
```

**PASS基準**: 全5項目がPASS

### Step 2: 第2層テスト — コンテンツテスト

各Phase Rulesファイルの内容品質を検証する。

**テスト項目**:

| テスト | 判定基準 | FAIL条件 |
|--------|---------|---------|
| 必須セクション存在 | 全ファイルに Purpose / Prerequisites / Execution Classification / Execution Steps / Output Artifacts / Completion Message / Error Handling が存在 | いずれかのセクション欠損 |
| Classification行の存在 | 全18ファイルに `ALWAYS EXECUTE` または `CONDITIONAL EXECUTE` が記載 | 1ファイルでも欠損 |
| GOOD/BAD例の件数 | 全ファイルに GOOD例 2件以上 + BAD例 2件以上 | GOOD/BAD例が2件未満のファイルが存在 |
| ドメイン特化率 | ファイルの内容に営業パイプライン固有の用語（商談/Opportunity/StageName/Salesforce/ガバナーリミット等）が本文の40%以上の段落に含まれる | ドメイン特化率 < 40% のファイルが存在 |
| Pitfall引用 | Error Handlingセクションで Domain Research 5 Pitfallsを少なくとも1つ引用 | Pitfall引用のないファイルが存在 |
| Completion Message形式 | 日本語で `**A) 修正依頼**` と `**B) [次ステージ]へ続行**` を含む | 形式不一致のファイルが存在 |

**ドメイン特化率の算出**:
```
domain_terms = ["商談", "Opportunity", "StageName", "Salesforce", "ガバナーリミット",
                "Bulk API", "Gmail", "Google Calendar", "Slack", "リード", "ステージ",
                "クロージング", "Activity", "MEDDIC", "BANT", "フォーキャスト",
                "パイプライン", "OAuth", "トークン"]

FOR each file IN phase_rules_files:
  paragraphs = split_into_paragraphs(file.content)
  domain_paragraphs = count(p for p in paragraphs if any(term in p for term in domain_terms))
  domain_rate = domain_paragraphs / len(paragraphs)

  IF domain_rate < 0.40:
    flag_fail(file, "ドメイン特化率不足", rate=domain_rate)
```

**PASS基準**: 全6項目が全18ファイルでPASS

### Step 3: 第3層テスト — スモークテスト

エージェントの実行フロー整合性を仮想的に検証する。

**テスト項目**:

| テスト | 判定基準 | FAIL条件 |
|--------|---------|---------|
| フロー整合性 | workflow-architecture.md のフロー図とPhase Rules の Prerequisites/Output Artifacts が一致 | S4のOutput ≠ A1のPrerequisites 等の不整合 |
| CONDITIONAL判定 | CONDITIONAL EXECUTEのファイルに Execute IF / Skip IF が明記されている | Execute IF / Skip IF が欠損 |
| CP参照整合性 | workflow-architecture.md のCP-1〜CP-8が対応するPhase Rulesに記載されている | CPが対応ファイルで未参照 |
| 繰り延べパス完全性 | D4の繰り延べ先（次サイクル）とS1の繰り延べリスト引き継ぎが整合 | 繰り延べアクションが次サイクルで失われるパスが存在 |

**フロー整合性チェック例**:
```
S4のOutput Artifacts:
  - 検出結果リスト（state/detection-results.yaml）

A1のPrerequisites:
  - S4の検出結果リスト（state/detection-results.yaml）が確定していること

→ 参照先ファイル名が一致: PASS ✓

D4のOutput Artifacts:
  - 繰り延べアクションリスト更新（state/deferred-actions.yaml）

S1のPrerequisites:
  - サイクル状態ファイルへの読み書き権限
  ※ state/deferred-actions.yaml の参照記載なし

→ 引き継ぎパスの記載不備: WARNING（FLAGとして記録、FAIL扱いにはしない）
```

**PASS基準**: 全4項目がPASS（WARNINGは記録するがPASS扱い）

### Step 4: テスト結果の集約と判定

3層テストの結果を集約し、最終判定を行う。

```yaml
validation_report:
  validated_at: "2026-04-05T15:00:00+09:00"
  plugin_name: "sales-pipeline-agent"
  version: "1.0.0"

  layer1_structure:
    status: PASS
    tests_passed: 5
    tests_failed: 0

  layer2_content:
    status: PASS
    files_validated: 18
    files_passed: 18
    files_failed: 0
    domain_rate_avg: 0.62

  layer3_smoke:
    status: PASS
    tests_passed: 4
    warnings: 1

  overall_status: PASS
  repair_attempts: 0
```

**全層PASS**: パッケージング完了 → 検証レポートを `logs/validation-{timestamp}.yaml` に保存
**いずれかFAIL**: P1へ戻るリペアループを実行

### Step 5: リペアループ制御（最大2回）

FAILの場合、P1へ戻って修正を実施する。

**リペアループルール**:
```
repair_count = 0
max_repairs = 2

WHILE overall_status == FAIL AND repair_count < max_repairs:
  repair_count += 1

  # FAILの内容に応じてリペア対象を特定
  IF layer1_fail:
    → P1の構造生成を再実行（欠損ファイル・パスの修正）
  IF layer2_fail:
    → 不合格ファイルのコンテンツを再生成（ドメイン特化率・例件数の補完）
  IF layer3_fail:
    → workflow-architecture.md との整合性修正

  re_run_p2()

IF repair_count >= max_repairs AND overall_status == FAIL:
  → リペアループ上限到達
  → 管理者に手動確認を要求
  → 検証失敗レポートを `logs/validation-failed-{timestamp}.yaml` に保存
```

---

## Output Artifacts

P2 が生成するアーティファクト:

1. **検証レポート**（`logs/validation-{timestamp}.yaml`） — パッケージング完了証跡
2. **検証失敗レポート**（リペアループ上限到達時） — 手動対応用
3. **パッケージング完了フラグ** — `plugin.json` に `validated: true` を追記

---

## Completion Message

P2 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — P2: Automated Validation 完了

検証結果（3層テスト）:
- 第1層（構造テスト）: [PASS / FAIL]
- 第2層（コンテンツテスト）: [PASS / FAIL]
  - ドメイン特化率平均: [X]%
  - Classification記載: [X]件 / 18件
  - GOOD/BAD例充足: [X]件 / 18件
- 第3層（スモークテスト）: [PASS / FAIL]

総合判定: [PASS / FAIL（リペア [N] 回実施）]
リペアループ残: [2-N]回

### WHAT'S NEXT
**A) 修正依頼** — 検証失敗の内容を手動修正する
**B) パッケージング完了** — プラグインの配布・使用開始が可能です
```

---

## Error Handling

### エラー 1: 第2層ドメイン特化率不足による品質低下

**症状**: Phase Rulesの一部が汎用的な説明のみで構成されており、営業パイプライン固有の用語・具体的な数値（ガバナーリミット閾値・停滞日数等）が不足している。エージェントが実際のドメイン課題に対して適切に動作しない。
**原因**: Phase Rules生成時にドメイン特化の指示が反映されなかった。
**対処**: FAILしたファイルを特定し、P1リペアでドメイン固有の内容（具体的な数値・API呼び出しパターン・Pitfall引用）を補完する。

**GOOD例**:
```
第2層テスト: schedule-check.md のドメイン特化率
→ 「Salesforce API使用率80%でセーフモード」「ガバナーリミット」「JST」「祝日カレンダー」等の用語を含む段落: 65%
→ ドメイン特化率 65% > 40% → PASS ✓
```

**BAD例**:
```
第2層テスト: schedule-check.md のドメイン特化率
→ ドメイン固有用語を含む段落: 25%（一般的な「APIチェック」「モード切り替え」の説明のみ）
→ ドメイン特化率 25% < 40% → FAIL
→ リペアループ1回目: Salesforce・ガバナーリミット・JST等の具体的な記述を補完して再生成
```

### エラー 2: Classification行欠損による実行制御不明確化

**症状**: 一部のPhase Rulesに `ALWAYS EXECUTE` / `CONDITIONAL EXECUTE` の記載がなく、エージェントがそのステージをスキップすべきかどうかを判断できない。不必要なステージが実行されたり、必須ステージがスキップされたりする。
**原因**: Phase Rules生成時にClassification行の記載が漏れた。
**対処**: 第2層テストのClassification検出で未記載ファイルを特定し、P1リペアで補完する。

**GOOD例**:
```
第2層テスト: 全18ファイルのClassification行確認
→ approval-request.md: "CONDITIONAL EXECUTE" 記載あり ✓
→ loop-control.md: "ALWAYS EXECUTE" 記載あり ✓
→ 18件 / 18件 PASS
```

**BAD例**:
```
第2層テスト: next-day-prep.md のClassification行確認
→ "CONDITIONAL EXECUTE" 記載なし → FAIL
→ エージェント実行時: 日中モードでもR2が実行され、不要な翌日準備処理が動作
→ ログに余分なエントリが記録され、リソースを無駄消費
```

### エラー 3: リペアループ上限到達後の無限修正試行

**症状**: リペアループを2回実施してもFAILが解消されず、3回目以降の修正を自動で試みる。無限修正ループに陥りビルドが永遠に完了しない。
**原因**: Step 5のリペアループ上限チェック（repair_count >= 2）が実装されていない。
**対処**: 2回のリペアでFAILが解消されない場合は「設計・コンテンツに根本的な問題がある」と判断し、自動修正を停止して管理者に手動確認を要求する。

**GOOD例**:
```
P2テスト: FAIL（layer2: ドメイン特化率不足3ファイル）
→ リペア1回目: 3ファイルを再生成 → テスト再実行
→ P2テスト: FAIL（layer2: 1ファイルがまだ不足）
→ リペア2回目: 残1ファイルを再生成 → テスト再実行
→ P2テスト: PASS → パッケージング完了
```

**BAD例**:
```
P2テスト: FAIL
→ リペア1回目: 修正 → FAIL
→ リペア2回目: 修正 → FAIL
→ リペア3回目（上限超過）: 修正試行
→ リペア10回目: まだFAIL
→ ビルドが24時間経っても完了しない
→ 管理者が気づかず、ポリシーの展開が永遠に遅延
```
