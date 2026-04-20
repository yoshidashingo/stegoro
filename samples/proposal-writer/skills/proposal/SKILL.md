---
name: proposal-writer
description: RFP対応提案書作成スキル。コンプライアンスマトリックス・技術提案・管理提案・価格フレームワークを含む包括的な提案書を自動生成するAnalytical型ステアリングポリシーのエントリポイント。
---

## Activation Trigger

以下のいずれかに該当する場合、このスキルを起動する。

- RFP（提案依頼書）を受け取り提案書作成を依頼された場合
- `/new-rfp` または `/resume-rfp` コマンドが実行された場合
- 「提案書を作成して」「RFPに応募したい」「入札書類を作って」等のキーワードが含まれる場合
- 既存提案書のレビューや品質チェックを依頼された場合

## Core Rules

### Rule 1: RFP完全読破（Pitfall 1対策）

提案書作成前にRFPの全ページを読み込み、コンプライアンスマトリックスを生成する。見落とした要件が1件でもある場合は要件抽出（A2）に戻る。「だいたい読んだ」は許容しない。

### Rule 2: 要件トレーサビリティ（Pitfall 2対策）

提案書の各セクションには対応するRFP要件番号を明示する。RFP参照なしの内容を生成した場合は即座にフラグを付与し、コンプライアンスチェック（R1）で検出できるようにする。

### Rule 3: 勝利戦略ファースト（Pitfall 3対策）

技術的な記述を開始する前に勝利戦略（P2: Win Strategy）を確定する。戦略なしで個別セクションを書き始めることは禁止。エグゼクティブサマリー（W1）は全セクション完成後に最後に書く。

### Rule 4: コンプライアンスゲート

最終アセンブリ（R3）前に必ずコンプライアンスチェック（R1）を実行し、全要件が対応済みであることを確認する。未対応要件が1件でも残る場合は提出不可。

### Rule 5: 品質スコアリング

品質レビュー（R2）では4項目（明確性・競争力・コンプライアンス・説得力）を各25点満点でスコアリングし、合計80点以上を提出基準とする。

## Workflow Summary

| フェーズ | ステージ | 分類 | 概要 |
|---------|---------|------|------|
| ANALYSIS | A1 RFP受付 | ALWAYS | RFP文書の構造把握・初期評価 |
| ANALYSIS | A2 要件抽出 | ALWAYS | 機能・非機能・コンプライアンス要件の抽出 |
| ANALYSIS | A3 コンプライアンスマトリックス | ALWAYS | 要件と提案セクションの対応表作成 |
| PLANNING | P1 提案構造設計 | ALWAYS | 提案書のアウトライン設計 |
| PLANNING | P2 勝利戦略 | ALWAYS | 差別化ポイント・評価者ニーズ分析 |
| PLANNING | P3 コンテンツ計画 | ALWAYS | セクション別執筆計画と担当割り当て |
| WRITING | W1 エグゼクティブサマリー | ALWAYS（最後） | 全セクション完成後に作成 |
| WRITING | W2 技術提案 | ALWAYS | ソリューション・アーキテクチャ・実装計画 |
| WRITING | W3 管理提案 | ALWAYS | プロジェクト管理・体制・リスク対応 |
| WRITING | W4 価格フレームワーク | CONDITIONAL | 価格提示が要求される場合 |
| REVIEW | R1 コンプライアンスチェック | ALWAYS | 全要件の対応状況を検証 |
| REVIEW | R2 品質レビュー | ALWAYS | 4項目スコアリング（目標80点以上） |
| REVIEW | R3 最終アセンブリ | ALWAYS | セクション統合・体裁統一・最終確認 |

## Quality Gate

最終アセンブリ（R3）前に以下の全項目がPASSであることを確認する。

| チェック | 基準 |
|---------|------|
| コンプライアンス | 全RFP要件がトレース済み（未対応0件） |
| 品質スコア | 4項目合計80点以上 |
| ページ制限 | RFP指定ページ制限内 |
| 価格根拠 | 全費用項目に根拠が記載済み |

## References

- `../../proposal-writer-rules/core-workflow.md` — ワークフロー全体定義
- `../../proposal-writer-rule-details/analysis/` — A1: rfp-intake, A2: requirements-extraction, A3: compliance-matrix
- `../../proposal-writer-rule-details/planning/` — P1: proposal-structure-design, P2: win-strategy, P3: content-planning
- `../../proposal-writer-rule-details/writing/` — W1: executive-summary, W2: technical-response, W3: management-response, W4: pricing-framework
- `../../proposal-writer-rule-details/review/` — R1: compliance-check, R2: quality-review, R3: final-assembly
- `../../proposal-writer-rule-details/common/` — process-overview, content-validation, overconfidence-prevention 他
