# Common Rules Design (v2 — Red Team指摘反映)

## Changes from v1
- **M5 fix**: `feedback-loop-and-retention.md` を新規追加（CSATフィードバックループ、保持期間、アクセス制御）
- tone-guidelines.md に感情ベースエスカレーション条件との連携を追記

## Common Rules Evaluation Matrix

| # | Rule | Include | Adaptation | Key Adaptation Points |
|---|------|---------|------------|----------------------|
| 1 | `welcome-message.md` | YES | Heavy | SaaSサポートエージェント紹介、3モード説明、対応カテゴリ一覧 |
| 2 | `process-overview.md` | YES | Heavy | 4運用フェーズ(TRIAGE→RESPONSE→QUALITY→FOLLOW-UP)+ビルドタイム(PACKAGING)のフロー図 |
| 3 | `question-format-guide.md` | YES | Light | 形式はSPM準拠。顧客への確認質問テンプレート追加 |
| 4 | `content-validation.md` | YES | Medium | PII検出パターン、禁止表現リスト、KB引用形式バリデーション |
| 5 | `session-continuity.md` | YES | Medium | チケット単位の状態管理（PIIフラグ、感情シグナル、KB信頼度スコアを追加） |
| 6 | `error-handling.md` | YES | Heavy | KB未ヒット/PII漏洩/エスカレーション先不在/SLA超過/トーン判定失敗+Q3リペアパスエラー |
| 7 | `overconfidence-prevention.md` | YES | Medium | ハルシネーション防止、エスカレーション保守的判定、セグメント誤判定防止 |
| 8 | `terminology.md` | YES | Heavy | Domain Research 8用語+追加用語（KB信頼度スコア、リペアパス、SLA超過警告等） |
| 9 | `quality-standards.md` | YES | Medium | 15品質次元をサポートドメインに適応 |
| 10 | `output-structure-patterns.md` | YES | Medium | メール回答/エスカレーションサマリー/傾向レポート/追跡記録のテンプレート |
| 11 | `tone-guidelines.md` (NEW v1) | YES | N/A | セグメント×感情トーンマトリクス。**v2追記**: 「即エスカレーション」セルはR1感情条件と連動 |
| 12 | `feedback-loop-and-retention.md` (NEW v2) | YES | N/A | CSATフィードバックループ、監査ログ保持期間、アクセス制御 |

## Domain-Specific Common Rules

### `tone-guidelines.md`（v1から継続 + v2追記）

内容はv1と同様。v2追記:

#### 感情ベースエスカレーション連携（v2 — C3 fix）
- トーンマトリクスで「即エスカレーション」と定義されたセルの条件は、R1 Mode Selection の感情ベースエスカレーション条件と**同一**である
- R1 で感情ベース判定を実施するため、通常はQ3到達前にエスカレーションが確定する
- R1で検出漏れした場合のセーフティネットとして、Q3でも同じ判定を行い、Q3→R4リペアパスで回収する

### `feedback-loop-and-retention.md`（v2新規 — M5 fix）

**Purpose**: 品質改善ループとコンプライアンス維持

**Content Outline**:

#### CSATフィードバックループ
- CSAT 3以下のチケットを週次レビュー対象に自動追加
- CSAT低評価パターンの分析（カテゴリ別/セグメント別/オペレーター別）
- 改善アクション: KB記事更新、トーンテンプレート修正、エスカレーション閾値調整

#### FAQ候補抽出との連携
- CSAT低評価 + 同カテゴリ反復問い合わせ → FAQ候補としてF2に投入
- 未整備KB領域の可視化

#### 監査ログ保持期間・アクセス制御
| データ種別 | 保持期間 | アクセス権限 |
|-----------|---------|-------------|
| チケットメタデータ（ID, カテゴリ, 優先度） | 3年 | サポートチーム全員 |
| 対応履歴（回答内容, エスカレーションサマリー） | 1年 | 担当者 + マネージャー |
| PIIを含む生データ（顧客メール原文） | 90日 | マネージャーのみ |
| 監査ログ（品質チェック結果） | 3年 | コンプライアンスチーム |
| CSATスコア・フィードバック | 1年 | サポートチーム全員 |

#### 削除ポリシー
- 保持期間経過後は自動パージ
- PIIを含むデータは保持期間内でも顧客の削除要求に応じる（GDPR準拠）

## Common Rules Summary

| File | Lines (Est.) | Adaptation Level |
|------|-------------|-----------------|
| welcome-message.md | 80-120 | Heavy |
| process-overview.md | 100-150 | Heavy |
| question-format-guide.md | 60-100 | Light |
| content-validation.md | 100-150 | Medium |
| session-continuity.md | 80-120 | Medium |
| error-handling.md | 150-200 | Heavy |
| overconfidence-prevention.md | 80-120 | Medium |
| terminology.md | 100-150 | Heavy |
| quality-standards.md | 120-180 | Medium |
| output-structure-patterns.md | 100-150 | Medium |
| tone-guidelines.md | 100-150 | N/A |
| feedback-loop-and-retention.md (NEW) | 100-150 | N/A |
| **Total** | **~1,170-1,640** | |
