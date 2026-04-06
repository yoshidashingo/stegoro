# Quality Mechanisms Design (v2 — Red Team指摘反映)

## Changes from v1
- **M1 fix**: ファイル数を統一（common 12 + phase rules 16 + core-workflow 1 = 29）
- **C1 fix**: Smoke Test に全モードがQUALITYフェーズを通過することの検証を追加
- **C2 fix**: Smoke Test に FOLLOW-UP フロー検証を追加
- **C3 fix**: Q3→R4 リペアパスの検証を追加
- **M3 fix**: SLA超過時挙動の検証を追加
- **M5 fix**: 保持期間・アクセス制御の検証を追加

## 15 Quality Dimensions — Customer Support Agent Mapping

### Standard Dimensions (Dim 1-11)

| # | Dimension | Target Agent Adaptation | Measurement |
|---|-----------|------------------------|-------------|
| 1 | Adaptive Workflow | 14運用stages: 11 ALWAYS + 3 CONDITIONAL (R4, R5, F1/F2)。モード判定(R1)でHybrid分岐 | 全ステージにALWAYS/CONDITIONAL分類あり |
| 2 | Mandatory Checkpoints | 8 CP: CP-1〜CP-8。全CPで2-option completion message | 全CPでREVIEW REQUIRED + WHAT'S NEXT |
| 3 | Question File Format | 顧客確認質問テンプレート、MCQ + Other + [Answer]: | 全質問ファイルがフォーマット準拠 |
| 4 | Content Validation | PII検出パターン、禁止表現リスト、KB引用形式、メール構造バリデーション | チェックリスト完備 |
| 5 | Audit Trail | チケット単位の監査ログ + PIIフラグ + 感情シグナル + KB信頼度 + CSAT | ISO 8601, append-only |
| 6 | Error Handling | 6シナリオ: KB未ヒット / PII漏洩 / エスカレーション先不在 / SLA超過 / トーン判定失敗 / Q3リペアパスエラー | 全フェーズ・全severity対応 |
| 7 | Overconfidence Prevention | ハルシネーション防止(KB外生成禁止), エスカレーション保守的判定(感情ベース追加), セグメント誤判定防止 | Red flags定義済み |
| 8 | Depth Levels | Minimal(単純FAQ, P4低) / Standard(技術問題, P3) / Comprehensive(複合問題・VIP・P1-P2) — T3の優先度で自動選択 | 3レベル+選択基準+ステージ適用ルール |
| 9 | Session Continuity | チケット状態ファイル（PIIフラグ/感情/KB信頼度/CSAT/SLA状態を追跡） | state file format定義 |
| 10 | Terminology Standardization | 12+用語のglossary（v1の8+KB信頼度スコア/リペアパス/SLA超過警告/営業時間等） | Common Misuse列付き |
| 11 | Standardized Completion Messages | 全ステージにREVIEW REQUIRED + WHAT'S NEXT (2 options) | No emergent patterns |

### Domain-Specific Dimensions (Dim 12-15)

| # | Dimension | Measurement Method | Threshold | Key Sources |
|---|-----------|-------------------|-----------|-------------|
| 12 | Domain Specificity Rate | サポート用語・SaaS手順・KB参照等のドメイン固有行 ÷ (総行数 - 空行 - 見出し行) | >= 40% | terminology.md, domain-research-summary |
| 13 | Example Coverage | 各Phase Ruleファイル内のGOOD/BADパターン数 | >= 2/file | 全16 Phase Ruleファイルに2例設計 |
| 14 | Artifact Template Completeness | テンプレートを含むべきファイル ÷ 実際にテンプレートがあるファイル | = 100% | R3(回答), R4(引き継ぎ+中間応答), R5(レポート), Q4(監査ログ), F1(追跡記録), F2(KB改善提案) = 6ファイル |
| 15 | Pitfall Reference Rate | error-handling内でDomain Research 4 Pitfallsを引用するアイテム ÷ 全エラーアイテム | >= 50% | Pitfall 1-4: ハルシネーション/PII漏洩/エスカレーション判定ミス/トーンミスマッチ |

## Checkpoint System

| CP | After Stage | Gate Content | Criteria |
|----|-------------|-------------|----------|
| CP-1 | T3: Priority Assessment | トリアージ結果確認 | カテゴリ・優先度・SLA確定 |
| CP-2 | R3: Draft Generation | 回答ドラフトレビュー | KB引用あり・構造OK |
| CP-3 | R4: Escalation Handoff | 引き継ぎサマリー確認 | コンテキスト完備 |
| CP-4 | R5: Trend Analysis | レポートレビュー | データ正確性確認 |
| CP-5 | Before Q4: Dispatch | 送信前最終確認 | PII clear + 正確 + トーンOK |
| CP-6 | F1: Case Tracking | フォローアップ状況確認 | 追跡計画・クローズ条件 |
| CP-7 | F2: FAQ Extraction | KB改善提案確認 | 候補の妥当性 |
| CP-8 | P1: Plugin Structure | プラグイン構造確認 | 構造・参照整合性 |

## 3-Layer Automated Testing

### Layer 1: Structure Test
| Check | Target | Pass Criteria |
|-------|--------|---------------|
| File existence | 全29ファイル (core 1 + common 12 + phase 16) | 全ファイル存在 |
| Cross-references | core-workflow.md内の全参照 | 100%解決 |
| Markdown validity | 全.mdファイル | 見出し階層・リスト・コードブロック正常 |
| Flow paths | 全フェーズの全パス | デッドエンドなし |
| File inventory consistency | 全設計ドキュメント間のファイル数 | 同一値 |

### Layer 2: Content Test
| Check | Target | Pass Criteria |
|-------|--------|---------------|
| Dim 12: Domain Specificity | 全16 Phase Ruleファイル | >= 40% per file |
| Dim 13: Example Coverage | 全16 Phase Ruleファイル | >= 2 GOOD/BAD per file |
| Dim 14: Artifact Templates | R3, R4, R5, Q4, F1, F2 | 100% (6/6) |
| Dim 15: Pitfall Reference | error-handling sections | >= 50% |
| Feedback loop coverage | common + quality rules | CSAT改善手順あり |
| Retention/access control | feedback-loop-and-retention.md | 保持期間・権限が明記 |
| Completion message format | 全ステージ | 2-option (Request Changes / Continue) |

### Layer 3: Smoke Test
| Check | Target | Pass Criteria |
|-------|--------|---------------|
| 即時回答フロー | T1→T2→T3→R1→R2→R3→Q1→Q2→Q3→Q4 | 全ステージ到達可能 |
| エスカレーションフロー（条件ベース） | T1→T2→T3→R1→R4→Q1→Q2→Q3→Q4 | 全ステージ到達可能、QUALITY通過 |
| エスカレーションフロー（KB未ヒット） | T1→T2→T3→R1→R2→R4→Q1→Q2→Q3→Q4 | KB信頼度<0.4でR4に切り替え |
| エスカレーションフロー（Q3リペアパス） | ...→Q3→R4→Q1→Q2→Q3→Q4 | リペアパス1回で収束 |
| 分析フロー | T1→T2→T3→R1→R5→Q1→Q2→Q3→Q4 | 全ステージ到達可能、QUALITY通過 |
| フォローアップフロー | Q4→F1→F2 | 条件付きで到達可能 |
| SLA超過パス | T3(Warning)→...→Q1(必須)→Q2(簡易)→Q3→Q4(超過フラグ) | PII Scanは省略不可 |
| PACKAGINGフロー | P1→P2 | 構造検証・スモークテスト完走 |
| Hybrid切り替え | R1で3モード+KB信頼度分岐+Q3リペア = 5パターンすべて分岐可能 | 条件明確・重複なし |

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
| Q3→R4 repair | Max 1 (2回目は強制エスカレーション) | リペアループ暴走防止 |
