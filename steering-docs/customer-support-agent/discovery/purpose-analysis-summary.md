# Purpose Analysis Summary

## Target Agent
- **Name**: customer-support-agent
- **Description**: SaaS/Webサービス向けカスタマーサポート対応エージェント。メールチャネルで問い合わせを受付し、状況に応じて即時回答・エスカレーション・分析モードを切り替えるハイブリッド型エージェント。

## Agent Classification
- **Type**: Hybrid Agent
- **Confidence**: High
- **Reasoning**: ユーザーが明示的にハイブリッド型を選択（Q1:D）。即時回答モード（FAQ・既知問題）、エスカレーションモード（複雑案件の人間引き継ぎ）、分析モード（問い合わせ傾向の把握）を状況に応じて切り替える。単一のProcess/Task/Analyticalパターンに収まらない。

### 4-Type Score
| Type | Score | Reasoning |
|------|-------|-----------|
| Process | 1 (Weak) | 順次プロセスではなく、モード切り替え型 |
| Task | 2 (Partial) | 個別の問い合わせ対応はタスク的だが、全体はそれ以上 |
| Analytical | 1 (Weak) | 分析モードはあるが主機能ではない |
| Hybrid | 3 (Strong) | 3つのモード切り替えが明確にHybridパターン |

## Core Capabilities

### Primary
- 問い合わせメールの受付・分類（カテゴリ判定、優先度判定）
- ナレッジベース参照による回答ドラフト生成
- 顧客セグメント別トーン制御（VIP / 一般 / クレーム対応）
- エスカレーション判定と人間オペレーターへの引き継ぎ

### Secondary
- 問い合わせ傾向の分析・レポート生成
- FAQ候補の自動抽出
- 対応履歴の要約・引き継ぎ情報生成

### Quality
- PII（個人情報）検出・マスキング
- 禁止表現チェック
- 回答正確性の検証（ナレッジベース照合）
- 監査ログの自動記録

### Interaction
- メール形式での回答生成（件名・本文・署名の構造化）
- エスカレーション時の引き継ぎサマリー生成
- 顧客への確認質問の生成

### Error
- ナレッジベースに該当情報がない場合のフォールバック
- PII検出漏れ時のセーフティネット
- エスカレーション先不在時の代替フロー

## Complexity Assessment
- **Overall**: Standard
- **Estimated Phases**: 3-4
- **Estimated Stages**: 10-14
- **Key Complexity Drivers**:
  - ハイブリッド型のモード切り替えロジック
  - 顧客セグメント別トーン制御
  - PII検出・マスキング要件
  - エスカレーション判定基準の設計

### Quantitative Indicators
| Indicator | Estimate |
|-----------|----------|
| Expected phases | 3-4 |
| Expected stages | 10-12 |
| Quality dimensions | 15 (all) |
| core-workflow lines | 300-450 |
| Total policy files | ~25 |

## Domain Profile
- **Domain**: SaaS / Webサービスのテクニカルサポート
- **Familiarity**: Well-known
- **Risk Level**: Medium（PII取り扱い、顧客満足度への直接影響）

## Open Questions
- 具体的なSaaS製品カテゴリ（プロジェクト管理、CRM、会計等）は Domain Research で深掘り
- ナレッジベースの形式・構造は Design Phase で詳細化
