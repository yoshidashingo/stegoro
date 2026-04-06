# Quality Mechanisms Design

## 15 Quality Dimensions — Customer Support Agent Mapping

### Standard Dimensions (Dim 1-11)

| # | Dimension | Target Agent Adaptation | Measurement |
|---|-----------|------------------------|-------------|
| 1 | Adaptive Workflow | 12 stages: 10 ALWAYS + 2 CONDITIONAL (R4, R5)。モード判定(R1)でHybrid分岐 | 全ステージにALWAYS/CONDITIONAL分類あり |
| 2 | Mandatory Checkpoints | 6 CP: CP-1(T3後), CP-2(R3後), CP-3(R4後), CP-4(R5後), CP-5(Q4後), CP-6(P1後) | 全CPで2-option completion message |
| 3 | Question File Format | 顧客への確認質問テンプレート（不足情報ヒアリング）、MCQ + Other + [Answer]: | 全質問ファイルがフォーマット準拠 |
| 4 | Content Validation | PII検出パターン、禁止表現リスト、KB引用形式、メール構造バリデーション | バリデーションチェックリスト完備 |
| 5 | Audit Trail | チケット単位の監査ログ: ID/timestamp/category/priority/mode/actions/quality-check-results | ISO 8601, append-only |
| 6 | Error Handling | 5シナリオ: KB未ヒット / PII漏洩 / エスカレーション先不在 / SLA超過 / トーン判定失敗 | 全フェーズ・全severity対応 |
| 7 | Overconfidence Prevention | ハルシネーション防止(KB外生成禁止), エスカレーション保守的判定, セグメント誤判定防止 | Red flags定義済み |
| 8 | Depth Levels | Minimal(単純FAQ) / Standard(技術問題) / Comprehensive(複合問題・VIP) | 3レベル定義+選択基準 |
| 9 | Session Continuity | チケット状態ファイルで対応途中の再開をサポート | state file format定義 |
| 10 | Terminology Standardization | 8+用語のglossary (チケット, エスカレーション, KB, SLA, CSAT, PII, FRT, トリアージ) | Common Misuse列付き |
| 11 | Standardized Completion Messages | 全ステージにREVIEW REQUIRED + WHAT'S NEXT (2 options) | No emergent patterns |

### Domain-Specific Dimensions (Dim 12-15)

| # | Dimension | Measurement Method | Threshold | Key Sources |
|---|-----------|-------------------|-----------|-------------|
| 12 | Domain Specificity Rate | サポート用語・SaaS手順・KB参照等のドメイン固有行 ÷ (総行数 - 空行 - 見出し行) | >= 40% | terminology.md, domain-research-summary |
| 13 | Example Coverage | 各Phase Ruleファイル内のGOOD/BADパターン数 | >= 2/file | phase-rules-design: 各ファイルに2例設計 |
| 14 | Artifact Template Completeness | テンプレートを含むべきファイル中、実際にテンプレートがあるファイルの割合 | = 100% | R3(回答テンプレート), R4(引き継ぎテンプレート), R5(レポートテンプレート), Q4(監査ログテンプレート) |
| 15 | Pitfall Reference Rate | error-handling内でDomain Research 4 Pitfallsを引用するアイテム ÷ 全エラーアイテム | >= 50% | Pitfall 1-4: ハルシネーション/PII漏洩/エスカレーション判定ミス/トーンミスマッチ |

## Checkpoint System

### Stage-Level Checkpoints

| CP | After Stage | Gate Content | Criteria |
|----|-------------|-------------|----------|
| CP-1 | T3: Priority Assessment | トリアージ結果確認（カテゴリ/優先度/SLA） | カテゴリ・優先度が確定 |
| CP-2 | R3: Draft Generation | 回答ドラフトレビュー（KB引用/構造/内容） | ドラフトが回答要件を満たす |
| CP-3 | R4: Escalation Handoff | 引き継ぎサマリー確認 | コンテキスト完備 |
| CP-4 | R5: Trend Analysis | 分析レポートレビュー | データ正確性確認 |
| CP-5 | Q4: Dispatch | 送信前最終確認（全品質チェックPASS） | PII clear + 正確 + トーンOK |
| CP-6 | P1: Plugin Structure | プラグイン構造確認 | 構造・参照整合性 |

## 3-Layer Automated Testing

### Layer 1: Structure Test
| Check | Target | Pass Criteria |
|-------|--------|---------------|
| File existence | 全25ファイル | 全ファイル存在 |
| Cross-references | core-workflow.md内の全参照 | 100%解決 |
| Markdown validity | 全.mdファイル | 見出し階層・リスト・コードブロック正常 |
| Flow paths | TRIAGE→RESPONSE→QUALITY の全パス | デッドエンドなし |

### Layer 2: Content Test
| Check | Target | Pass Criteria |
|-------|--------|---------------|
| Dim 12: Domain Specificity | 全Phase Ruleファイル | >= 40% per file |
| Dim 13: Example Coverage | 全Phase Ruleファイル | >= 2 GOOD/BAD per file |
| Dim 14: Artifact Templates | R3, R4, R5, Q4 | 100% (4/4) |
| Dim 15: Pitfall Reference | error-handling sections | >= 50% |

### Layer 3: Smoke Test
| Check | Target | Pass Criteria |
|-------|--------|---------------|
| 即時回答フロー | T1→T2→T3→R1→R2→R3→Q1→Q2→Q3→Q4 | 全ステージ到達可能 |
| エスカレーションフロー | T1→T2→T3→R1→R4 | 全ステージ到達可能 |
| 分析フロー | T1→T2→T3→R1→R5 | 全ステージ到達可能 |
| Hybrid切り替え | R1で3モードすべて分岐可能 | 条件明確・重複なし |

## Repair Judgment Tree — Full Mapping

| Dimension | FAIL Classification | Return Target |
|-----------|-------------------|---------------|
| Dim 1: Adaptive Workflow | Design | Workflow Architecture |
| Dim 2: Mandatory Checkpoints | Structural | Core Workflow Gen |
| Dim 3: Question File Format | Content | Common Rules Gen |
| Dim 4: Content Validation | Content | Common Rules Gen |
| Dim 5: Audit Trail | Content | Common Rules Gen |
| Dim 6: Error Handling | Content | Common Rules Gen / Phase Rules Gen |
| Dim 7: Overconfidence Prevention | Content | Common Rules Gen |
| Dim 8: Depth Levels | Design | Workflow Architecture |
| Dim 9: Session Continuity | Content | Common Rules Gen |
| Dim 10: Terminology | Content | Common Rules Gen |
| Dim 11: Completion Messages | Structural | Core Workflow Gen |
| Dim 12: Domain Specificity | Content | Phase Rules Gen |
| Dim 13: Example Coverage | Content | Phase Rules Gen |
| Dim 14: Artifact Templates | Content | Phase Rules Gen |
| Dim 15: Pitfall Reference | Content | Phase Rules Gen |

## Loop Control Rules

| Rule | Value | Rationale |
|------|-------|-----------|
| Max retries (total) | 3 | コンテキスト爆発防止 |
| Same-target limit | 2 | 問題分類誤りの示唆 |
| Escalation options | Continue / Abort / Rescope | ユーザー制御の確保 |
| P2→P1 loop | Max 2 (separate counter) | プラグイン修正は限定的 |
