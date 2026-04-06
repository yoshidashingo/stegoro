# Scope Definition

## Target Agent Summary
- **Name**: customer-support-agent
- **Type**: Hybrid Agent
- **Domain**: SaaS / Webサービスのテクニカルサポート（メール）
- **Complexity**: Standard

## Scope Boundaries

### In Scope
| Area | Coverage | Notes |
|------|----------|-------|
| Workflow | メール問い合わせの受付→トリアージ→対応（即時回答/エスカレーション/分析）→品質検査→送信→フォローアップ | 3モード切り替え型 |
| Quality | 15品質次元すべて。PII検出・マスキング、禁止表現チェック、回答正確性検証、監査ログ | GDPR/個人情報保護法準拠 |
| Users | サポートオペレーター（エージェントの利用者）。間接的に顧客（エンドユーザー） | 単一ユーザータイプ |
| Domain | SaaS製品のテクニカルサポート。アカウント/課金/機能/バグ/セキュリティの各カテゴリ | 汎用SaaS（特定製品に非依存） |
| Errors | ハルシネーション防止、PII漏洩防止、エスカレーション判定ミス、トーンミスマッチ | Domain Researchの4 Pitfalls |
| Integration | ナレッジベース参照、チケット管理システム連携、SLAモニタリング | 具体的APIは実装時に確定 |

### Out of Scope
- **チャット/電話チャネル**: メールのみ対応（将来拡張候補）
- **多言語対応**: 日本語単一言語（将来拡張候補）
- **KB自動生成**: KBの参照はスコープ内だが、KB記事の自動生成・更新は対象外
- **顧客セルフサービスポータル**: エージェントはオペレーター向け、顧客直接対応UIは対象外
- **リアルタイム感情分析モデル**: ルールベースの感情シグナル検出のみ（ML推論は対象外）

## Estimated Structure

### Phase Structure
- **Phases**: 4（TRIAGE, RESPONSE, QUALITY, PACKAGING）
- **Total Stages**: 12（ALWAYS: 10, CONDITIONAL: 2）
- **Common Rule Files**: 11（standard: 10 + domain-specific: 1 tone-guidelines.md）
- **Phase Rule Files**: 12
- **Total Files**: ~25（core-workflow 1 + common 11 + phase rules 12 + plugin files ~1）
- **Estimated Total Lines**: ~3,500-4,500

### Phase 1: TRIAGE（トリアージ）
- Stage 1: Ticket Reception（チケット受付）(ALWAYS)
- Stage 2: Classification（分類・カテゴリ判定）(ALWAYS)
- Stage 3: Priority Assessment（優先度・SLA判定）(ALWAYS)

### Phase 2: RESPONSE（対応）
- Stage 4: Mode Selection（モード判定: 即時回答/エスカレーション/分析）(ALWAYS)
- Stage 5: Knowledge Retrieval（ナレッジベース検索）(ALWAYS)
- Stage 6: Draft Generation（回答ドラフト生成）(ALWAYS)
- Stage 7: Escalation Handoff（エスカレーション引き継ぎ）(CONDITIONAL — Execute IF: エスカレーション判定)
- Stage 8: Trend Analysis（傾向分析・レポート生成）(CONDITIONAL — Execute IF: 分析モード判定)

### Phase 3: QUALITY（品質検査）
- Stage 9: PII Scan（PII検出・マスキング）(ALWAYS)
- Stage 10: Content Review（正確性・禁止表現チェック）(ALWAYS)
- Stage 11: Tone Calibration（トーン適合性チェック）(ALWAYS)
- Stage 12: Dispatch（送信・監査ログ記録）(ALWAYS)

### Phase 4: PACKAGING
- Plugin Structure Generation (ALWAYS)
- Automated Validation (ALWAYS)

## Quality Target
- **Level**: Standard
- **15 Dimensions Target**: 15/15 PASS
- **Key Quality Focus Areas**:
  - Dim 12: Domain Specificity Rate >= 40%（SaaSサポート用語・手順の反映）
  - Dim 13: Example Coverage >= 2/file（GOOD/BADの具体的対応例）
  - Dim 15: Pitfall Reference Rate >= 50%（Domain Researchの4 Pitfalls引用）

## Directory Structure

```text
.customer-support-agent/
├── customer-support-agent-rules/
│   └── core-workflow.md
└── customer-support-agent-rule-details/
    ├── common/
    │   ├── welcome-message.md
    │   ├── process-overview.md
    │   ├── question-format-guide.md
    │   ├── content-validation.md
    │   ├── session-continuity.md
    │   ├── error-handling.md
    │   ├── overconfidence-prevention.md
    │   ├── terminology.md
    │   ├── quality-standards.md
    │   ├── output-structure-patterns.md
    │   └── tone-guidelines.md
    ├── triage/
    │   ├── ticket-reception.md
    │   ├── classification.md
    │   └── priority-assessment.md
    ├── response/
    │   ├── mode-selection.md
    │   ├── knowledge-retrieval.md
    │   ├── draft-generation.md
    │   ├── escalation-handoff.md
    │   └── trend-analysis.md
    ├── quality/
    │   ├── pii-scan.md
    │   ├── content-review.md
    │   ├── tone-calibration.md
    │   └── dispatch.md
    └── packaging/
        ├── plugin-structure-generation.md
        └── automated-validation.md
```

## Effort Estimate
- **DESIGN Phase**: Workflow Architecture + Common/Phase Rules Design + Quality Mechanisms — 4 stages
- **GENERATION Phase**: Core Workflow + Common Rules(11) + Phase Rules(12) + Integration Validation — 4 stages
- **REFINEMENT Phase**: Completeness + Consistency + Quality Calibration — 3 stages
- **PACKAGING Phase**: Plugin Structure + Automated Validation — 2 stages
