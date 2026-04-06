# Phase Rules Design

## Phase Rule Files Inventory

### Phase 1: TRIAGE（3 files）

#### T1: `triage/ticket-reception.md`
- **Classification**: ALWAYS
- **Purpose**: メール解析、顧客情報抽出、チケットオブジェクト生成
- **Content Outline**:
  - メールヘッダー解析（From, Subject, In-Reply-To for thread detection）
  - 顧客情報ルックアップ（セグメント判定: VIP/一般、プラン: Free/Pro/Enterprise）
  - 重複チケット検出（同一顧客からの24h以内の類似問い合わせ）
  - チケットオブジェクト生成テンプレート
- **Domain Injection Points**: 顧客セグメント判定ロジック、SaaS製品カテゴリリスト
- **Examples**: GOOD — 正しいセグメント判定例、BAD — スレッド検出漏れで重複チケット作成
- **Est. Lines**: 150-200

#### T2: `triage/classification.md`
- **Classification**: ALWAYS
- **Purpose**: 問い合わせカテゴリ・サブカテゴリの自動判定
- **Content Outline**:
  - カテゴリ体系: アカウント / 課金 / 機能 / バグ / セキュリティ
  - サブカテゴリ（各カテゴリに3-5個）
  - キーワードマッチング + コンテキスト分析のハイブリッド分類
  - 複数カテゴリ該当時の優先ルール（セキュリティ > バグ > 課金 > 機能 > アカウント）
  - 分類信頼度スコアと低信頼時のフォールバック
- **Domain Injection Points**: SaaS固有のカテゴリ体系、キーワード辞書
- **Examples**: GOOD — セキュリティ関連の正確な分類、BAD — 課金問題をアカウント問題に誤分類
- **Est. Lines**: 150-200

#### T3: `triage/priority-assessment.md`
- **Classification**: ALWAYS
- **Purpose**: 優先度スコアリングとSLA残時間算出
- **Content Outline**:
  - 優先度スコア計算式: セグメント重み × カテゴリ重み × 緊急度指標
  - SLAテーブル（プラン別: Enterprise 2h/Pro 8h/Free 24h FRT）
  - 優先度レベル: P1(Critical) / P2(High) / P3(Medium) / P4(Low)
  - SLA残時間のリアルタイム計算とアラート閾値
- **Domain Injection Points**: SLA数値、優先度計算の重み付け
- **Examples**: GOOD — Enterprise顧客のセキュリティ問題→P1判定、BAD — SLA計算で営業時間を考慮せず
- **Est. Lines**: 150-200

### Phase 2: RESPONSE（5 files）

#### R1: `response/mode-selection.md`
- **Classification**: ALWAYS
- **Purpose**: Hybridモード判定（即時回答/エスカレーション/分析）
- **Content Outline**:
  - エスカレーション条件チェック（最優先判定）:
    - セキュリティインシデント / データ損失リスク / 法的問題
    - 顧客の明示的オペレーター要求
    - 3往復以上未解決
  - 分析モード条件: 管理者リクエスト / 定期レポートタイミング
  - デフォルト: 即時回答モード
  - 判定ツリーのフロー図
- **Domain Injection Points**: エスカレーション条件の具体的閾値
- **Examples**: GOOD — データ損失リスク検出→即エスカレーション、BAD — 顧客の「上司を出せ」を見逃し
- **Est. Lines**: 150-200

#### R2: `response/knowledge-retrieval.md`
- **Classification**: ALWAYS
- **Purpose**: ナレッジベース検索と関連情報の取得
- **Content Outline**:
  - KB検索戦略: キーワード検索 → セマンティック検索 → カテゴリフィルタリング
  - 検索結果のランキング（関連度 × 鮮度 × 信頼度）
  - ヒットなし時のフォールバック: 類似記事提示 → 「確認の上回答」テンプレート
  - 過去の類似チケット解決事例の参照
  - **ハルシネーション防止**: KB外情報の生成を明示的に禁止
- **Domain Injection Points**: KB検索API仕様、検索結果の信頼度閾値
- **Examples**: GOOD — 複数KB記事を統合した包括的回答素材、BAD — 古いKB記事を最新として引用
- **Est. Lines**: 150-250

#### R3: `response/draft-generation.md`
- **Classification**: ALWAYS
- **Purpose**: メール回答ドラフトの生成
- **Content Outline**:
  - 回答構造: 挨拶 → 問題理解の確認 → 解決策/回答 → 次のステップ → 署名
  - KB引用ルール: 回答の根拠となるKB記事URLを必ず含める
  - パーソナライズ: 顧客名、プラン名、問い合わせ履歴の参照
  - マルチステップ回答: 手順が複数ある場合の番号付きリスト形式
  - 「確認の上回答」テンプレート（KB未ヒット時）
- **Domain Injection Points**: メールテンプレート、署名形式、KB記事参照形式
- **Examples**: GOOD — KB引用付きの構造化回答、BAD — KB未ヒットなのに推測で手順を案内
- **Est. Lines**: 200-250

#### R4: `response/escalation-handoff.md`
- **Classification**: CONDITIONAL
- **Execute IF**: R1でエスカレーションモード判定
- **Skip IF**: 即時回答モードまたは分析モード
- **Purpose**: 人間オペレーターへの的確な引き継ぎ
- **Content Outline**:
  - 引き継ぎサマリーテンプレート: 顧客情報/問い合わせ要約/試行済み対応/推奨対応/緊急度
  - エスカレーション先の選定（カテゴリ別担当チーム）
  - 顧客への中間応答テンプレート（「担当者に引き継ぎます」）
  - エスカレーション後の追跡: 解決確認までのフォローアップ
- **Domain Injection Points**: エスカレーション先チーム定義、SLA引き継ぎルール
- **Examples**: GOOD — 完全なコンテキスト付き引き継ぎ、BAD — 対応履歴なしで引き継ぎ→顧客に再説明要求
- **Est. Lines**: 150-200

#### R5: `response/trend-analysis.md`
- **Classification**: CONDITIONAL
- **Execute IF**: 分析モード判定（管理者リクエスト/定期レポート）
- **Skip IF**: 即時回答モードまたはエスカレーションモード
- **Purpose**: 問い合わせ傾向の分析とレポート生成
- **Content Outline**:
  - 集計軸: カテゴリ別/優先度別/セグメント別/期間別
  - KPIダッシュボード: FRT, 解決率, CSAT, エスカレーション率
  - トレンド検出: 急増カテゴリ、繰り返し問い合わせパターン
  - レポートテンプレート: サマリー → 詳細 → 推奨アクション
- **Domain Injection Points**: KPI定義、レポート形式、閾値設定
- **Examples**: GOOD — 課金関連問い合わせ急増→値上げ通知との相関特定、BAD — データ不足で誤ったトレンド報告
- **Est. Lines**: 150-200

### Phase 3: QUALITY（4 files）

#### Q1: `quality/pii-scan.md`
- **Classification**: ALWAYS
- **Purpose**: 回答ドラフト内のPII検出と自動マスキング
- **Content Outline**:
  - PIIパターン定義（正規表現）: メールアドレス/電話番号/クレジットカード/マイナンバー/住所/IPアドレス
  - マスキングルール: 検出→置換（`***@***.com`形式）
  - False positive対応: 製品名やURLに含まれる数字列の除外
  - 検出ログ: 検出箇所・パターン・処置の記録
  - **二重検査**: 入力時（T1）+ 出力時（Q1）の2段階
- **Domain Injection Points**: PIIパターン正規表現、マスキング形式
- **Examples**: GOOD — カード番号を正しくマスキング、BAD — URLのクエリパラメータ内のメールアドレスを見逃し
- **Est. Lines**: 150-200

#### Q2: `quality/content-review.md`
- **Classification**: ALWAYS
- **Purpose**: 回答の正確性と禁止表現チェック
- **Content Outline**:
  - KB照合: 回答内の手順・URL・機能名がKBと一致するか検証
  - 禁止表現スキャン: ネガティブフレーズリスト（tone-guidelines.md参照）
  - 事実確認: バージョン番号・価格・機能有無の最新情報照合
  - 不正確検出時のフロー: 自動修正可能→修正 / 不可能→フラグ付きで人間レビュー
- **Domain Injection Points**: 禁止表現リスト、KB照合API
- **Examples**: GOOD — 廃止済み機能への言及を検出→代替案に差し替え、BAD — 旧料金プランの価格を案内
- **Est. Lines**: 150-200

#### Q3: `quality/tone-calibration.md`
- **Classification**: ALWAYS
- **Purpose**: 顧客セグメント×感情に基づくトーン適合性チェック
- **Content Outline**:
  - tone-guidelines.md のトーンマトリクスとの照合
  - セグメント判定結果（T1）と感情シグナル検出結果の統合
  - トーン不適合時の自動調整: フレーズ置換、敬語レベル調整
  - クレーム対応時の特別ルール: 共感→謝罪→解決策の3ステップ確認
- **Domain Injection Points**: トーンマトリクス、フレーズ置換テーブル
- **Examples**: GOOD — 怒りシグナル検出→共感表現追加、BAD — VIP顧客にテンプレート的応対
- **Est. Lines**: 120-180

#### Q4: `quality/dispatch.md`
- **Classification**: ALWAYS
- **Purpose**: 最終確認、メール送信、監査ログ記録
- **Content Outline**:
  - 最終チェックリスト: PII clear / 正確性 verified / トーン calibrated / KB引用 present
  - 送信形式: 件名ルール、本文構造、署名
  - 監査ログ記録: チケットID/タイムスタンプ/カテゴリ/優先度/モード/処理時間/品質チェック結果
  - CSAT依頼の自動付与（設定に応じて）
  - SLA達成/未達の記録
- **Domain Injection Points**: メール送信API仕様、監査ログ形式、CSAT調査テンプレート
- **Examples**: GOOD — 全チェック通過→送信→監査ログ完備、BAD — PII scanスキップで個人情報含む回答を送信
- **Est. Lines**: 120-180

### Phase 4: PACKAGING（2 files）

#### P1: `packaging/plugin-structure-generation.md`
- **Classification**: ALWAYS
- **Purpose**: Claude Codeプラグイン構造の生成
- **Est. Lines**: 100-150

#### P2: `packaging/automated-validation.md`
- **Classification**: ALWAYS
- **Purpose**: 3層テスト（構造+コンテンツ+スモーク）
- **Est. Lines**: 100-150

## File Inventory Summary

| Phase | Files | Est. Lines |
|-------|-------|-----------|
| TRIAGE | 3 | 450-600 |
| RESPONSE | 5 | 800-1,100 |
| QUALITY | 4 | 540-760 |
| PACKAGING | 2 | 200-300 |
| **Total Phase Rules** | **14** | **~1,990-2,760** |

## Inter-Stage Dependencies

```
T1 → T2 → T3 → R1
                 ├─→ R2 → R3 → Q1 → Q2 → Q3 → Q4
                 ├─→ R4 (CONDITIONAL)
                 └─→ R5 (CONDITIONAL)
```

## Domain Content Injection Points Summary

| Category | Injection Target | Source |
|----------|-----------------|--------|
| カテゴリ体系 | T2, R1, R5 | Domain Research: 分類ベストプラクティス |
| SLA数値 | T3, Q4 | Domain Research: ITIL/SLA framework |
| PIIパターン | Q1, content-validation.md | Domain Research: GDPR/個人情報保護法 |
| トーンルール | Q3, tone-guidelines.md | Domain Research: トーン制御ベストプラクティス |
| KB検索戦略 | R2, R3 | Domain Research: ハルシネーション防止 |
| エスカレーション条件 | R1, R4 | Domain Research: エスカレーションベストプラクティス |
