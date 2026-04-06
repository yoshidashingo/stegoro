# Phase Rules Design (v2 — Red Team指摘反映)

## Changes from v1
- **C1 fix**: R4/R5 → Q1-Q4 を通過するフローに変更。各ファイルの記述を更新
- **C2 fix**: FOLLOW-UP フェーズ（F1, F2）を新設
- **C3 fix**: R1 に感情ベースエスカレーション条件追加。Q3 にリペアパス追記
- **M1 fix**: ファイル数を16に統一（TRIAGE 3 + RESPONSE 5 + QUALITY 4 + FOLLOW-UP 2 + PACKAGING 2）
- **M3 fix**: T3 にSLA超過警告、Q 各ステージにSLA短縮パス記述追加
- **M4 fix**: R2 にKB信頼度スコア閾値とエスカレーション切り替え追加

## Phase Rule Files Inventory

### Phase 1: TRIAGE（3 files）

#### T1: `triage/ticket-reception.md`
- **Classification**: ALWAYS
- **Purpose**: メール解析、顧客情報抽出、PII初期検出・フラグ付与、チケット生成
- **Content Outline**:
  - メールヘッダー解析（From, Subject, In-Reply-To for thread detection）
  - 顧客情報ルックアップ（セグメント判定: VIP/一般、プラン: Free/Pro/Enterprise）
  - **PII初期検出・フラグ付与（v2）**: 入力メールをスキャンし、検出PIIの種別（メール/電話/カード等）をチケットにフラグとして記録。この時点ではマスキングせず、出力時（Q1）でマスキング
  - 重複チケット検出（同一顧客からの24h以内の類似問い合わせ）→ 検出時: 既存チケットに追記し、反復回数をインクリメント
  - **感情シグナル初期検出（v2）**: 怒り/不満/困惑/通常を判定し、チケットに記録。R1で参照
  - チケットオブジェクト生成テンプレート
- **Domain Injection Points**: 顧客セグメント判定ロジック、PIIパターン正規表現、感情シグナルキーワード
- **Examples**: GOOD — PII 3種（メール/カード/電話）を正しくフラグ付与、BAD — スレッド検出漏れで重複チケット作成
- **Est. Lines**: 180-230

#### T2: `triage/classification.md`
- **Classification**: ALWAYS
- **Purpose**: 問い合わせカテゴリ・サブカテゴリの自動判定
- **Content Outline**: v1と同様（カテゴリ体系、キーワード+コンテキスト分類、優先ルール、信頼度スコア）
- **Examples**: GOOD — セキュリティ関連の正確な分類、BAD — 課金問題をアカウント問題に誤分類
- **Est. Lines**: 150-200

#### T3: `triage/priority-assessment.md`
- **Classification**: ALWAYS
- **Purpose**: 優先度スコアリング、SLA算出、SLA超過警告
- **Content Outline**:
  - 優先度スコア計算式: セグメント重み × カテゴリ重み × 緊急度指標 × **反復係数（v2: 反復回数2以上で1.5倍）**
  - SLAテーブル（プラン別: Enterprise 2h/Pro 8h/Free 24h FRT）
  - **セキュリティインシデントSLA（v2）**: プラン問わず1h FRT
  - 優先度レベル: P1(Critical) / P2(High) / P3(Medium) / P4(Low)
  - **SLA超過警告（v2 — M3 fix）**: SLA残15分でWarning、超過でExceeded → State Trackingに記録、管理者自動通知
  - 営業時間定義: タイムゾーン（JST）、営業日（月-金）、祝日カレンダー参照
- **Examples**: GOOD — Enterprise顧客のセキュリティ問題→P1+1h SLA、BAD — SLA計算で営業時間を考慮せず
- **Est. Lines**: 180-230

### Phase 2: RESPONSE（5 files）

#### R1: `response/mode-selection.md`
- **Classification**: ALWAYS
- **Purpose**: Hybridモード判定（即時回答/エスカレーション/分析）
- **Content Outline**:
  - **エスカレーション条件チェック（v2 — C3 fix: 感情ベース追加）**:
    - 条件ベース: セキュリティインシデント / データ損失リスク / 法的問題 / 顧客の明示的オペレーター要求 / 同一チケット3往復以上未解決 or チケット横断で同カテゴリ3件以上
    - **感情ベース（v2 NEW）**: T1で検出された感情シグナル「怒り」 × セグメント「クレーム対応中」→ 即エスカレーション。tone-guidelines.md のトーンマトリクスと同一条件
  - 分析モード条件: 管理者リクエスト / 定期レポートタイミング
  - デフォルト: 即時回答モード
  - 判定ツリーのフロー図
- **Examples**: GOOD — 怒りシグナル+クレーム対応中→即エスカレーション判定、BAD — 「担当者を出せ」を見逃し即時回答
- **Est. Lines**: 180-230

#### R2: `response/knowledge-retrieval.md`
- **Classification**: ALWAYS
- **Purpose**: KB検索、KB信頼度スコア算出、閾値判定
- **Content Outline**:
  - KB検索戦略: キーワード検索 → セマンティック検索 → カテゴリフィルタリング
  - 検索結果のランキング（関連度 × 鮮度 × 信頼度）
  - **KB信頼度スコアと閾値（v2 — M4 fix）**:
    - スコア >= 0.6: 即時回答モード続行 → R3へ
    - スコア 0.4-0.6: 「確認の上回答」テンプレート使用 → R3へ
    - スコア < 0.4: エスカレーションモードに切り替え → R4へ
  - **ハルシネーション防止**: KB外情報の生成を明示的に禁止
  - 過去の類似チケット解決事例の参照（反復問い合わせ時は過去の対応経緯を統合）
- **Examples**: GOOD — 信頼度0.3でエスカレーション切り替え、BAD — 信頼度0.2でも「確認の上回答」テンプレートを使用
- **Est. Lines**: 180-250

#### R3: `response/draft-generation.md`
- **Classification**: ALWAYS
- **Purpose**: メール回答ドラフトの生成
- **Content Outline**: v1と同様（回答構造、KB引用、パーソナライズ）
  - **v2追記**: クレーム対応中セグメント向けドラフト構造: 共感パラグラフ → 過去対応経緯の確認 → 謝罪 → 解決策 → 次のステップ（Q3前にドラフト段階で構造対応）
- **Examples**: GOOD — KB引用+クレーム対応構造のドラフト、BAD — KB未ヒットなのに推測で手順を案内
- **Est. Lines**: 200-250

#### R4: `response/escalation-handoff.md`
- **Classification**: CONDITIONAL
- **Execute IF**: R1でエスカレーション判定 / R2でKB信頼度<0.4 / Q3でリペアパス
- **Skip IF**: 即時回答モード続行 and 分析モード
- **Purpose**: 人間オペレーターへの引き継ぎ + 中間応答ドラフト作成
- **Content Outline**:
  - 引き継ぎサマリーテンプレート
  - エスカレーション先の選定（カテゴリ別担当チーム）
  - **中間応答ドラフト作成（v2 — C1 fix）**: 「担当者に引き継ぎます」テンプレートをドラフトとして作成。**Q1-Q4を通過してからの送信**
  - エスカレーション後の追跡: F1（Case Tracking）に引き継ぎ
- **Examples**: GOOD — 完全なコンテキスト付き引き継ぎ+PIIマスキング済み中間応答、BAD — 中間応答にPII含有のまま送信
- **Est. Lines**: 180-230

#### R5: `response/trend-analysis.md`
- **Classification**: CONDITIONAL
- **Execute IF**: 管理者が分析モードをリクエスト / 定期レポートタイミング
- **Purpose**: 問い合わせ傾向の分析とレポート生成
- **Content Outline**: v1と同様（集計軸、KPI、トレンド検出）
  - **v2追記**: レポートドラフトは**Q1-Q4を通過**（PIIがレポートに混入していないか検査）
- **Examples**: GOOD — 課金問い合わせ急増トレンド検出+PII除去済みレポート、BAD — 顧客メールアドレスがレポートに残存
- **Est. Lines**: 150-200

### Phase 3: QUALITY（4 files）

#### Q1: `quality/pii-scan.md`
- **Classification**: ALWAYS
- **Purpose**: 全モードの出力に対するPII検出・マスキング（v2: 全モード共通化）
- **Content Outline**:
  - PIIパターン定義（正規表現）: メールアドレス/電話番号/クレジットカード/マイナンバー/住所/IPアドレス
  - **社内スタッフPIIの処理（v2）**: 社内メールアドレス・内線番号もPII対象。マスキングルールは同一
  - マスキングルール: 検出→置換
  - False positive対応
  - 検出ログ: 検出箇所・パターン・処置の記録
  - **適用対象（v2 — C1 fix）**: 即時回答ドラフト / エスカレーション中間応答+引き継ぎサマリー / 分析レポート
  - **SLA超過時もQ1は省略不可（v2 — M3 fix）**
- **Examples**: GOOD — 引き継ぎサマリー内のカード番号をマスキング、BAD — 社内スタッフのメールアドレスを見逃し
- **Est. Lines**: 170-220

#### Q2: `quality/content-review.md`
- **Classification**: ALWAYS
- **Purpose**: 正確性・禁止表現チェック
- **Content Outline**: v1と同様
  - **v2追記**: SLA残15分未満時は簡易チェック（KB照合のみ、事実確認はスキップ可）
  - **v2追記**: KB記事の鮮度再検証（R2で取得した記事の最終更新日が90日以上前の場合、警告フラグ）
- **Examples**: GOOD — 廃止済み機能への言及を検出→代替案に差し替え、BAD — 90日以上更新されていないKB記事を最新として引用
- **Est. Lines**: 160-210

#### Q3: `quality/tone-calibration.md`
- **Classification**: ALWAYS
- **Purpose**: トーン適合性チェック + エスカレーション判定リペアパス
- **Content Outline**:
  - tone-guidelines.md のトーンマトリクスとの照合
  - トーン不適合時の自動調整
  - クレーム対応時の3ステップ確認（共感→謝罪→解決策）
  - **リペアパス（v2 — C3 fix）**:
    - トーンマトリクスで「即エスカレーション」判定 → 現在モードが即時回答の場合、R4に遷移
    - R4で中間応答を生成し、再びQ1から品質検査を通過
    - リペアは最大1回。2回目の即エスカレーション判定は強制エスカレーション（Q4で処理）
  - **対象（v2 — C1 fix）**: 即時回答ドラフト / エスカレーション中間応答 / 分析レポート
- **Examples**: GOOD — 怒りシグナル検出→Q3でリペアパス発動→R4にエスカレーション、BAD — VIP顧客にテンプレート的応対を見逃し
- **Est. Lines**: 150-200

#### Q4: `quality/dispatch.md`
- **Classification**: ALWAYS
- **Purpose**: 最終確認、送信/引き継ぎ/レポート出力、監査ログ記録
- **Content Outline**:
  - **CP-5: 送信前最終確認（v2: Q4の前に移動）**
  - 最終チェックリスト: PII clear / 正確性 verified / トーン calibrated / KB引用 present
  - **モード別出力先（v2 — C1 fix）**:
    - 即時回答: メール送信
    - エスカレーション: 中間応答メール送信 + 引き継ぎサマリーをオペレーターに送付
    - 分析: レポートを管理者に送付
  - 監査ログ記録
  - **SLA達成/未達記録 + 超過時の管理者通知（v2 — M3 fix）**
  - CSAT依頼の自動付与
- **Examples**: GOOD — 全チェックPASS+SLA達成+監査ログ完備、BAD — PII scanスキップで個人情報含む回答を送信
- **Est. Lines**: 150-200

### Phase 4: FOLLOW-UP（2 files — v2新設 — C2 fix）

#### F1: `follow-up/case-tracking.md`
- **Classification**: CONDITIONAL
- **Execute IF**: エスカレーション済み / 保留回答送信後 / 顧客返信待ち
- **Skip IF**: 即時回答で解決済み、追加連絡なし
- **Purpose**: 未解決案件の追跡、SLA再監視、クローズ条件チェック
- **Content Outline**:
  - チケット管理システムへの状態更新（Open/Pending/Escalated/Resolved/Closed）
  - 次回アクション日時の記録
  - SLA再計算（エスカレーション解決SLA: オペレーター引き継ぎ後4h）
  - クローズ条件: 顧客確認済み / 7日間返信なし / オペレーターが解決報告
  - **チケット管理システム連携の責務定義（v2）**: create(T1) / update(R1,Q4) / escalate(R4) / resolve(F1) / close(F1)
- **Examples**: GOOD — エスカレーション48h後に解決確認→クローズ、BAD — エスカレーション後放置で顧客から再問い合わせ
- **Est. Lines**: 150-200

#### F2: `follow-up/faq-candidate-extraction.md`
- **Classification**: CONDITIONAL
- **Execute IF**: 同カテゴリ問い合わせが週5件以上 / CSAT 3以下が3件蓄積
- **Skip IF**: 通常運用範囲内
- **Purpose**: FAQ候補の抽出とKB改善提案
- **Content Outline**:
  - 重複問い合わせクラスタリング（カテゴリ×サブカテゴリ×キーワード）
  - 未整備KB領域の特定
  - CSAT低評価との相関分析
  - KB改善提案テンプレート: 問題概要 / 推奨KB記事内容 / 影響チケット数 / 優先度
- **Examples**: GOOD — 「請求書ダウンロード」問い合わせ週8件→FAQ記事作成提案、BAD — サンプル数不足で誤ったトレンド報告
- **Est. Lines**: 120-170

### Build-Time: PACKAGING（2 files）

#### P1: `packaging/plugin-structure-generation.md`
- **Classification**: ALWAYS
- **Purpose**: Claude Codeプラグイン構造の生成
- **Content Outline**: plugin.json, marketplace.json, agent定義, skill定義, command定義の生成手順
- **Est. Lines**: 100-150

#### P2: `packaging/automated-validation.md`
- **Classification**: ALWAYS
- **Purpose**: 3層テスト（構造+コンテンツ+スモーク）
- **Content Outline**: テスト定義、PASS/FAIL基準、P2→P1ループ制御
- **Est. Lines**: 100-150

## File Inventory Summary

| Phase | Files | Est. Lines |
|-------|-------|-----------|
| TRIAGE | 3 | 510-660 |
| RESPONSE | 5 | 890-1,160 |
| QUALITY | 4 | 630-830 |
| FOLLOW-UP (v2 NEW) | 2 | 270-370 |
| PACKAGING | 2 | 200-300 |
| **Total Phase Rules** | **16** | **~2,500-3,320** |

## Inter-Stage Dependencies (v2)

```
T1 → T2 → T3 → R1
                 ├─→ R2 ──┬─ (信頼度>=0.6) → R3 → Q1 → Q2 → Q3 → Q4
                 │         └─ (信頼度<0.4)  → R4 → Q1 → Q2 → Q3 → Q4
                 ├─→ R4 → Q1 → Q2 → Q3 → Q4
                 └─→ R5 → Q1 → Q2 → Q3 → Q4
                                        ↑
                              Q3 リペアパス → R4 (max 1回)

Q4 → F1 (CONDITIONAL) → F2 (CONDITIONAL)
```
