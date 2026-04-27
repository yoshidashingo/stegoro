---
name: proposal-writer:validate-policy
description: 生成済み提案書のコンプライアンスと品質を検証する
---

## Execution

1. 対象の提案書ファイルまたはディレクトリを指定する
2. `rule-details/review/compliance-check.md` を読み込む
3. コンプライアンスチェック（R1）を実行: 全RFP要件のトレース確認
4. `rule-details/review/quality-review.md` を読み込む
5. 品質レビュー（R2）を実行: 4項目スコアリング
6. 検証レポートを生成して出力する

## Prerequisites

- RFP文書が参照可能であること
- 対象提案書が存在すること

## Output

- 検証レポート: コンプライアンス状況、品質スコア、改善提案
