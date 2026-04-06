# Common Rules Design

## Common Rules Evaluation Matrix

| # | Rule | Include | Adaptation | Key Adaptation Points |
|---|------|---------|------------|----------------------|
| 1 | `welcome-message.md` | YES | Heavy | SaaSサポートエージェントとして自己紹介、3モード（即時回答/エスカレーション/分析）の説明、対応可能カテゴリ一覧 |
| 2 | `process-overview.md` | YES | Heavy | 4フェーズ(TRIAGE→RESPONSE→QUALITY→PACKAGING)のフロー図、Hybridモード切り替えの説明 |
| 3 | `question-format-guide.md` | YES | Light | 形式はSPM準拠。顧客への確認質問テンプレート追加（不足情報ヒアリング用） |
| 4 | `content-validation.md` | YES | Medium | PII検出パターン（メール/電話/カード番号/住所）、禁止表現リスト、KB引用形式のバリデーション追加 |
| 5 | `session-continuity.md` | YES | Medium | チケット単位の状態管理（チケットID/顧客/カテゴリ/優先度/モード/現在ステージ） |
| 6 | `error-handling.md` | YES | Heavy | KB検索ヒットなし、PII検出漏れ、エスカレーション先不在、SLA超過、トーン判定失敗の各シナリオ |
| 7 | `overconfidence-prevention.md` | YES | Medium | ハルシネーション防止（KB外情報の生成禁止）、エスカレーション判定の保守的運用、セグメント誤判定防止 |
| 8 | `terminology.md` | YES | Heavy | Domain Researchの8用語＋追加用語（トリアージ、セグメント、CSAT、FRT、SLA、KB、PII、エスカレーション等） |
| 9 | `quality-standards.md` | YES | Medium | 15品質次元をサポートドメインに適応。Dim 12: サポート用語/手順の反映率、Dim 15: 4 Pitfalls引用率 |
| 10 | `output-structure-patterns.md` | YES | Medium | メール回答テンプレート、エスカレーションサマリーテンプレート、傾向レポートテンプレートの構造パターン |

## Domain-Specific Common Rule

### `tone-guidelines.md`（新規追加）

**Purpose**: 顧客セグメント×感情シグナルに基づくトーン制御ルール

**Content Outline**:

#### トーンマトリクス

| Segment \ Emotion | 通常 | 困惑 | 不満 | 怒り |
|-------------------|------|------|------|------|
| **VIP** | 丁寧+迅速対応約束 | 共感+優先対応明示 | 深い謝罪+即エスカ検討 | 即エスカレーション |
| **一般** | 標準丁寧語 | 共感+手順案内 | 謝罪+解決策提示 | 謝罪+エスカ検討 |
| **クレーム対応中** | 共感+経過報告 | 傾聴+詳細確認 | 深い謝罪+代替案複数 | 即エスカレーション |

#### ネガティブフレーズ変換テーブル

| NG表現 | OK表現 |
|--------|--------|
| できません | 代替案をご提案いたします |
| わかりません | 確認の上、改めてご連絡いたします |
| お客様の誤りです | 操作手順をご案内いたします |
| 仕様です | 現在の動作仕様についてご説明いたします |

#### 感情シグナル検出ルール
- **怒り**: 「！」連続、大文字強調、「ふざけるな」「いい加減にしろ」等
- **不満**: 「何度も」「まだ」「いつまで」等の反復・期限表現
- **困惑**: 「わからない」「どうすれば」「困っている」等
- **通常**: 上記シグナルなし

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
| tone-guidelines.md (NEW) | 100-150 | N/A (新規) |
| **Total** | **~1,070-1,490** | |
