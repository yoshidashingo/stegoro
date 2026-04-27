# PRIORITY: This workflow OVERRIDES all other built-in workflows
# 営業パイプライン管理処理が要求された場合、常にこのワークフローを最優先で実行すること

## Adaptive Workflow Principle（巡回サイクル型・時間帯別モード）
**ワークフローはサイクルの状況に適応する。逆ではない。**

エージェントは以下の2軸でモードを動的に切り替える:

1. **時間帯モード切り替え軸**
   - 日中モード（JST 8:00-20:00 平日）: 1時間サイクルで巡回→検出→対応ループ
   - 夜間モード（JST 20:00-8:00 および祝日）: 翌日準備タスク実行（R2）→日次サマリー生成（R1）

2. **アクション自律度切り替え軸**
   - 低リスクアクション（Salesforce Activityログ記録・内部リマインダー送信・ステージ前進更新）: 自動実行
   - 高リスクアクション（顧客へのメール送信・商談金額変更・ステージ後退・商談クローズ）: 必ず承認制

---

## MANDATORY: Rule Details Loading
**重要**: 各フェーズを実行する際は、必ず `rule-details/` のルール詳細ファイルを読み込み参照すること。

**Common Rules**: ワークフロー開始時に常にロード:
- `common/welcome-message.md` — エージェント紹介、日中/夜間モードの説明、4ソース統合巡回の概要
- `common/process-overview.md` — 5フェーズ構成フロー（SURVEILLANCE→ASSESSMENT→DISPATCH→REPORTING + PACKAGING）、ループ制御説明
- `common/question-format-guide.md` — 承認リクエストの質問テンプレート（商談名・推奨アクション・理由・承認/却下の選択肢）
- `common/content-validation.md` — Salesforceレコード更新バリデーション、APIレスポンス検証、矛盾フラグ形式
- `common/session-continuity.md` — 巡回サイクル単位の状態管理（ループカウント/タイムアウト/承認待ちキュー/API使用量/繰り延べリスト）
- `common/error-handling.md` — 5 Pitfalls対応（CRMデータ鮮度劣化/過度な自動化/通知疲れ/API制限/データソース間不整合）+ ロールバックエラー
- `common/overconfidence-prevention.md` — 停滞判定の過信防止、自動実行/承認制境界の保守的判定、商談金額・ステージ変更への慎重な姿勢
- `common/terminology.md` — 12ドメイン用語（パイプライン/商談/リード/ステージ/クロージング日/アクティビティ/加重パイプライン/BANT/フォーキャスト/SFA/ガバナーリミット/パイプラインカバレッジ）+ エージェント固有5用語
- `common/quality-standards.md` — 15品質次元のパイプライン管理適応版
- `common/output-structure-patterns.md` — 日次サマリー/承認リクエスト/自動実行通知/ロールバック通知/監査ログのテンプレート
- `common/action-authority.md` — 自動実行/承認制の境界定義（低リスク/高リスクの分類基準、例外ケース、タイムアウトルール）
- `common/data-source-config.md` — 4ソースの接続設定・OAuth管理・APIレート制限・フォールバック設定

## MANDATORY: Action Authority（自動実行/承認制の境界）
**重要**: いかなる状況でも `common/action-authority.md` の境界定義に従うこと。

**自動実行（低リスク）**:
- Salesforce Activityログ記録（Gmail送受信・カレンダーミーティング・Slack言及の自動記録）
- 内部リマインダー送信（担当営業へのSlack DM）
- ステージの前進更新（Prospecting→Qualification 等、後退でない場合のみ）
- 会議準備サマリー生成（ミーティング24時間前）
- パイプラインヘルスレポート生成（読み取り専用）

**承認制（高リスク）**:
- 顧客へのメール送信（Gmail経由の外部送信）
- 商談金額の変更（Opportunityの金額フィールド）
- ステージの後退（Negotiation→Proposal 等）
- 商談クローズ（Won/Lost）
- 新規リードの商談化
- 既存レコードの削除・重複レコードのマージ

**境界が曖昧な場合は常に承認制に分類すること（Pitfall 2対策）**

---

# 営業パイプライン管理エージェント — 巡回サイクル型ワークフロー

---

# SURVEILLANCE PHASE（Phase 1）

**目的**: 4データソース（Salesforce / Gmail / Google Calendar / Slack）を定期巡回し、対応要件を持つ変更・異常を検出する
**フォーカス**: データ収集の完全性・正確性、統合ビューの構築
**ステージ**: S1（ALWAYS）, S2（ALWAYS）, S3（ALWAYS）, S4（ALWAYS）

---

## S1: Schedule Check（ALWAYS EXECUTE）

詳細ルール: `surveillance/schedule-check.md` を参照

1. 現在時刻（JST）を取得し、時間帯モードを判定する
   - **日中モード**: JST 8:00-20:00 かつ平日（祝日カレンダー参照）
   - **夜間モード**: JST 20:00-8:00 または祝日
2. 前回サイクルの完了状態を確認する
   - 前回が45分タイムアウトで強制終了していた場合: 繰り延べリストを引き継ぎ、現サイクルの優先アクションに追加する
   - 繰り延べリストが存在する場合: サイクル状態ファイルに記録し、A2（Action Identification）で参照可能にする
3. Salesforce API使用量を確認する（Pitfall 4対策）
   - 使用率 < 80%: 通常モード継続（1時間巡回サイクル）
   - 使用率 80-95%: **セーフモード移行**（巡回間隔を2時間に延長、管理者にSlack通知）
   - 使用率 >= 95%: **読み取り専用モード**（DML操作を停止、巡回のみ続行）
4. セーフモード中の承認済みアクション: Salesforceへの書き込みを保留し次サイクルへ繰り延べる
5. **夜間モード時のダイレクトパス**: S2→S3→S4をスキップしてR2（Next-Day Prep）へ直行する
6. サイクル状態ファイルにモード・API使用量・開始タイムスタンプを記録する
7. **Wait for Explicit Approval**: スケジュール状態と動作モードを提示する（CP-1）
8. **MANDATORY**: S1結果を監査ログに記録する

## S2: Source Connection（ALWAYS EXECUTE — 日中モードのみ）

詳細ルール: `surveillance/source-connection.md` を参照

1. 4ソースの接続確認を並行実行する（Salesforce / Gmail / Google Calendar / Slack）
2. 各ソースのOAuthアクセストークン有効期限を確認する
   - 残り30分以内: リフレッシュトークンでアクセストークンを自動更新する
   - リフレッシュトークン失効: 当該ソースを「障害」フラグで記録し、次のステージへ続行する
3. 部分障害時の続行判断（Pitfall 4対策）:
   - Salesforceのみ障害: 承認制アクションを停止し読み取り専用で続行する
   - Gmail/Calendar/Slackのみ障害: 当該ソース由来の検出カテゴリをスキップして続行する
   - 全ソース障害: 巡回をスキップし、障害通知を最終通知チャンネルに送信してサイクルを終了する
4. 接続状態（connected / auth_error / rate_limit / partial）をサイクル状態ファイルに記録する
5. `common/data-source-config.md` のフォールバック優先順位に従って障害ソースの扱いを決定する
6. **MANDATORY**: 接続状態を監査ログに記録する

## S3: Data Collection（ALWAYS EXECUTE — 日中モードのみ）

詳細ルール: `surveillance/data-collection.md` を参照

1. **差分取得**: 前回巡回タイムスタンプ以降の変更データのみを取得する（全量取得は禁止）
2. 各ソースから以下のデータを収集する:
   - **Salesforce**: 商談（Opportunity）のステージ・金額・クロージング日・最終更新日・Activity履歴・リード一覧
   - **Gmail**: 送受信メール（Subject / From / To / Date / Snippet）・スレッドID
   - **Google Calendar**: 外部参加者を含むミーティング予定（今後14日間）
   - **Slack**: 指定チャンネルの商談名・顧客名を含むメッセージ（前回巡回以降）
3. **統合ビュー構築**: タイムスタンプベースで最新情報を優先にマージする
4. **ソース間矛盾検出**（Pitfall 5対策）:
   - Salesforceステージ「Closed」かつGmailで直近7日以内に商談継続メール → 矛盾フラグを付与する
   - Salesforce Activity未記録かつGmail/Calendar/Slackでアクティビティあり → ステータス未反映フラグを付与する
   - 矛盾フラグ付き案件はASSESSMENTで人間判断を仰ぐ（自動判定は禁止）
5. **Salesforce Bulk API 2.0活用**（Pitfall 4対策）: 50件以上のレコード取得はバルクAPIで一括取得する。SOQLクエリは必要フィールドのみ SELECT すること（SELECT * は禁止）
6. 障害ソースは統合ビューの当該項目を「データなし」として扱い、レポートに障害ソースを明記する
7. **MANDATORY**: 収集件数・矛盾フラグ件数を監査ログに記録する

## S4: Change Detection（ALWAYS EXECUTE — 日中モードのみ）

詳細ルール: `surveillance/change-detection.md` を参照

1. 統合ビューから以下の7種の変更・異常を検出する:

   **検出ルール1 — 停滞商談**（Pitfall 1対策: 多層判定必須）:
   - Salesforceで閾値超え（Proposalステージ: 14日、Negotiationステージ: 7日、その他: 21日）
   - **かつ** Gmail/Slackで直近N日以内にアクティビティなし（Nは閾値の半分）
   - 矛盾フラグ付き案件は停滞自動検出の対象外とする（人間判断に委ねる）

   **検出ルール2 — 未返信メール**:
   - 担当者送信メールへの返信が3営業日超過
   - 既にリマインダー送信済みの場合は重複通知を抑制する（Pitfall 3対策）

   **検出ルール3 — 会議準備**:
   - Google Calendarで外部参加者ありの予定が24時間以内
   - かつ対応するSalesforce商談が存在する

   **検出ルール4 — 金額変動**:
   - Salesforce Opportunityの金額フィールドが前回巡回比で10%以上変動している

   **検出ルール5 — 期限接近**:
   - クロージング日まで7日以内（緊急: 1日以内）
   - 既に通知済みの場合は48時間以内の重複通知を抑制する（Pitfall 3対策）

   **検出ルール6 — 未追跡リード**:
   - Gmail/CalendarのやりとりでSalesforceにリード/商談が存在しない外部ドメインの連絡先

   **検出ルール7 — ステータス未反映**:
   - Gmail/Calendar/SlackでActivityがあるが、SalesforceのActivityログに記録がない

2. 検出結果を優先度で分類する:
   - **緊急**: クロージング日1日以内の商談の異変、大型商談（¥10M以上）の停滞検出
   - **重要**: 停滞商談検出・未返信フォロー必要
   - **情報**: Activity自動記録対象・ステータス更新対象
3. 同一商談に複数ルールが同時適用された場合は統合して1件として扱う（1商談1検出原則、Pitfall 3対策）
4. 変更検出リストを確定してASSESSMENT PHASEへ渡す
5. 検出なし（0件）の場合: ASSESSMENTをスキップしてR1（Daily Summary Generation）へ直行する
6. **Wait for Explicit Approval**: 変更検出リスト（件数・種別・多層判定結果）を提示する（CP-2）
7. **MANDATORY**: 検出件数・種別・優先度を監査ログに記録する

---

# ASSESSMENT PHASE（Phase 2）

**目的**: 検出された変更・異常を評価し、対応アクションを特定、優先度付け、自動実行/承認制の分類を行う
**フォーカス**: 正確な商談ヘルス評価と、過不足ない対応アクションの特定
**ステージ**: A1（ALWAYS）, A2（ALWAYS）, A3（ALWAYS）, A4（ALWAYS）

---

## A1: Deal Health Scoring（ALWAYS EXECUTE）

詳細ルール: `assessment/deal-health-scoring.md` を参照

1. S4で検出された各商談のヘルススコアを算出する（0-100）
2. スコア算出インプット:
   - ステージ滞留日数（滞留長いほどスコア低下）
   - 最終アクティビティからの経過日数（時間が経つほどスコア低下）
   - メール応答率（応答率が高いほどスコア上昇）
   - ミーティング頻度（頻度が高いほどスコア上昇）
   - クロージング日までの残日数（近いほど緊急度上昇）
   - 商談金額（大型商談は重みを増加）
3. スコア区分:
   - 赤（0-39）: 緊急対応が必要
   - 黄（40-69）: 注意が必要、フォローアップ推奨
   - 緑（70-100）: 健全
4. **多層判定によるスコア補正**（Pitfall 5対策）: Salesforceで停滞スコア低でも、Gmail/Slackで直近アクティビティがあればスコアを補正する。Salesforceデータのみによる単一指標判定は禁止
5. 矛盾フラグ付き案件はスコア算出を保留し「人間判断要」としてフラグを維持する
6. **MANDATORY**: ヘルススコアと算出根拠を監査ログに記録する

## A2: Action Identification（ALWAYS EXECUTE）

詳細ルール: `assessment/action-identification.md` を参照

1. S4の7種検出結果とA1のヘルススコアに基づき、具体的な対応アクションを生成する:

   | 検出種別 | 生成アクション |
   |---------|-------------|
   | 停滞商談 | Salesforce Activityログ更新提案、担当者フォローアップリマインダー |
   | 未返信メール | フォローアップメール下書き生成（承認制）、リマインダー送信（自動） |
   | 会議準備 | 商談サマリー自動生成（Salesforce情報・直近メール・過去議事録）（自動） |
   | 金額変動 | 変動通知生成、承認後の金額更新 |
   | 期限接近 | 担当者への緊急アラート送信（Slack DM）（自動）、クロージング準備チェックリスト生成 |
   | 未追跡リード | Salesforceへのリード作成提案（承認制） |
   | ステータス未反映 | Salesforce Activityログ自動記録（自動）、ステージ更新提案（承認制） |

2. 前サイクルから繰り延べられたアクションを優先リストの先頭に追加する
3. 矛盾フラグ付き案件には「ソース間矛盾 — 人間判断要」のアクションを生成する
4. **MANDATORY**: 生成アクションリストを監査ログに記録する

## A3: Priority Ranking（ALWAYS EXECUTE）

詳細ルール: `assessment/priority-ranking.md` を参照

1. 各アクションの優先度スコアを算出する:
   - **優先度スコア** = 商談金額（正規化）× 期限リスク（残日数逆数）× ヘルススコア逆数（リスク高いほど優先）× 繰り延べフラグ（繰り越し案件は1.5倍）
2. 優先度レベルの分類:
   - **P1（Critical）**: 優先度スコア上位20%、または緊急通知対象（クロージング日1日以内・¥10M以上の赤スコア商談）
   - **P2（High）**: 優先度スコア上位21-40%、または黄スコアの期限接近商談
   - **P3（Medium）**: 優先度スコア上位41-70%
   - **P4（Low）**: 残り
3. P1アクションはDISPATCH PHASEで最優先処理する
4. **MANDATORY**: 優先度付きアクションリストを監査ログに記録する

## A4: Authority Classification（ALWAYS EXECUTE）

詳細ルール: `assessment/authority-classification.md` および `common/action-authority.md` を参照

1. 各アクションを `common/action-authority.md` の基準に従い分類する:
   - **自動実行リスト**: 低リスクアクション（Activity記録・内部リマインダー・会議準備サマリー・ステージ前進更新）
   - **承認制リスト**: 高リスクアクション（メール送信・金額変更・ステージ後退・商談クローズ・リード商談化・削除）
   - **保留リスト**: 矛盾フラグ付き案件（人間判断が必要なもの）
2. 境界が曖昧な場合は常に承認制に分類する（Pitfall 2対策）
3. 夜間モード中の顧客向けアクション: 承認済みであっても翌営業日8:00まで実行を保留リストに移す
4. 読み取り専用モード中（Salesforce API 95%超）: 自動実行リストのSalesforce書き込みアクションを保留リストへ移す
5. アクションリストが空（自動実行も承認制もなし）の場合: D4（Loop Control）へ直行する
6. **Wait for Explicit Approval**: アクション分類結果（自動実行/承認制/保留の内訳）を提示する（CP-3）
7. **MANDATORY**: 分類済みアクションリストを監査ログに記録する

---

# DISPATCH PHASE（Phase 3）

**目的**: 分類されたアクションを実行し、ループ制御によって残件がなくなるまで対応を繰り返す
**フォーカス**: 安全な自動実行と承認フローの適切な管理
**ステージ**: D1（CONDITIONAL）, D2（CONDITIONAL）, D3（CONDITIONAL）, D4（ALWAYS）, D5（ALWAYS）

---

## D1: Auto-Execute（CONDITIONAL EXECUTE）

**実行条件**: A4の自動実行リストに1件以上のアクションが存在する場合
**スキップ条件**: 自動実行対象がゼロ、または読み取り専用モード中

詳細ルール: `dispatch/auto-execute.md` を参照

1. **排他制御**: 同一商談レコードへの並行更新を防ぐため、商談IDレベルのロックを取得してから実行する
2. 優先度（P1→P2→P3→P4）の順に自動実行アクションを処理する
3. 各アクションの実行前に事前状態（Salesforceレコードの現在値）をスナップショットとして保存する（ロールバック用）
4. Salesforce APIのバルクAPI活用（Pitfall 4対策）: 複数レコードへの書き込みはバッチ処理でまとめて実行する
5. 実行結果（成功/失敗）をサイクル状態ファイルに即時記録する
6. 実行失敗時: エラー内容を監査ログに記録し、当該アクションを承認制リストへ昇格させる（自動リトライは禁止）
7. 全自動実行完了後、D4（Loop Control）へ進む
8. **MANDATORY**: 実行アクション・事前状態スナップショット・実行結果を監査ログに記録する

## D2: Approval Request（CONDITIONAL EXECUTE）

**実行条件**: A4の承認制リストに1件以上のアクションが存在する場合
**スキップ条件**: 承認制対象がゼロ

詳細ルール: `dispatch/approval-request.md` を参照

1. `common/output-structure-patterns.md` の承認リクエストテンプレートを使用して承認リクエストを生成する
2. 承認リクエストの内容（1件ごと）:
   - 商談名・担当者名・アクション種別
   - 推奨アクションの詳細と実行理由
   - 関連するヘルススコア・優先度・検出種別
   - 承認/却下の選択肢
3. P1アクションは担当者へSlack DM（即時）で送信する
4. P2-P4アクションはバッチ送信（同一担当者への通知は1通にまとめる、Pitfall 3対策）
5. **タイムアウト管理**:
   - 送信後30分: 自動リマインダーをSlack DMで送信する
   - 送信後2時間: 未承認の場合、当該アクションを次サイクルへ繰り延べる（エージェントをブロックしない）
6. 承認リクエスト送信記録（商談ID・送信時刻・タイムアウト時刻）をサイクル状態ファイルに記録する
7. D4（Loop Control）へ進む（承認を待たずに続行）
8. **MANDATORY**: 承認リクエスト内容・送信記録を監査ログに記録する

## D3: Approval Processing（CONDITIONAL EXECUTE）

**実行条件**: 承認済みアクション（前回サイクルからの繰り越し承認含む）が存在する場合
**スキップ条件**: 承認済みアクションがゼロ

詳細ルール: `dispatch/approval-processing.md` を参照

1. 承認結果（承認/却下）を取得する（Slack応答またはシステムからのコールバック）
2. **承認の場合**:
   - 夜間制約の確認: 夜間モード中の顧客向けアクションは翌営業日8:00まで保留する
   - 承認内容と実行予定の一致を確認する
   - 実行前の事前状態スナップショットを保存する
   - アクションを実行する
3. **却下の場合**:
   - 却下理由を監査ログに記録する
   - 同種アクションの自動判定ロジックに「見直しフラグ」を立てる
   - 当該アクションを次サイクルへ持ち越さない（再提案は不可）
4. **Wait for Explicit Approval**: 承認済みアクションの実行前に内容確認を提示する（CP-4）
5. 実行結果をサイクル状態ファイルに記録する
6. D4（Loop Control）へ進む
7. **MANDATORY**: 承認者・承認時刻・実行結果を監査ログに記録する

## D4: Loop Control（ALWAYS EXECUTE）

詳細ルール: `dispatch/loop-control.md` を参照

1. **残件チェック**: 対応未完了のアクション（未実行の自動実行・未承認の承認制アクション）を確認する
2. **ループ継続条件**: 以下をすべて満たす場合、A1（Deal Health Scoring）へ戻り次のループを開始する
   - 残件アクション数 > 0
   - 現サイクルのループカウント < 5（最大5ループ）
   - 現サイクルの経過時間 < 45分（タイムアウト）
3. **ループ終了条件**: 以下のいずれかに該当する場合、D5（Rollback Check）へ進む
   - 残件アクション数 = 0（全対応完了）
   - ループカウントが5に達した（上限到達 → 残件は次サイクルへ繰り延べ）
   - 経過時間が45分を超過した（タイムアウト → 残件は次サイクルへ繰り延べ）
4. **新規検出の扱い**（Pitfall 3対策）: ループ中に新たに検出された案件は現サイクルのアクションリストに追加しない。次サイクルへ繰り延べる
5. 繰り延べアクションをサイクル状態ファイルの「繰り延べリスト」に記録する
6. ループカウントをサイクル状態ファイルに更新する
7. **MANDATORY**: ループ判定結果（継続/終了理由）を監査ログに記録する

## D5: Rollback Check（ALWAYS EXECUTE）

詳細ルール: `dispatch/rollback-check.md` を参照

1. 本サイクルで自動実行したSalesforceレコード変更の事後状態を確認する
2. D1で保存した事前状態スナップショットと比較する
3. **異常検出の基準**:
   - ステージが意図しない方向に変更されている（前進のつもりが後退）
   - 金額フィールドが承認なく変更されている
   - 削除操作が未承認で実行されている
4. **異常検出時の対応**:
   - ロールバックAPIを呼び出して事前状態に復元する
   - 担当者にSlack通知を送信する（ロールバック内容・原因・確認依頼）
   - ロールバック実行記録を監査ログに記録する
5. **ロールバック失敗時**: 管理者に緊急通知を送信し、手動修正ガイドを提示する。当該アクションの自動実行を一時停止フラグを立てる
6. 全チェック完了後（異常なし or ロールバック完了）、REPORTING PHASEへ進む
7. **Wait for Explicit Approval**: ロールバックチェック結果を提示する（CP-5）
8. **MANDATORY**: ロールバックチェック結果（OK/ロールバック実行/失敗）を監査ログに記録する

---

# REPORTING PHASE（Phase 4）

**目的**: サイクル結果を集約し、日次サマリーを配信、監査ログを確定する
**フォーカス**: 情報の正確な集約とコンパクトな通知（Pitfall 3: 通知疲れ防止）
**ステージ**: R1（ALWAYS）, R2（CONDITIONAL）, R3（ALWAYS）

---

## R1: Daily Summary Generation（ALWAYS EXECUTE）

詳細ルール: `reporting/daily-summary-generation.md` を参照

1. 本サイクルの対応状況を集約する:
   - 自動実行完了件数・内訳
   - 承認制アクション件数（承認済み/却下/承認待ち/繰り延べ）
   - 未対応残件数・繰り延べリスト件数
   - パイプラインヘルスサマリー（赤/黄/緑の件数分布）
2. **通知の階層化**（Pitfall 3対策）:
   - **緊急**（Slack DM即時）: クロージング日24時間以内の商談の異変、大型商談（¥10M以上）の停滞検出
   - **重要**（次回巡回サマリーに集約）: 停滞商談検出、未返信フォロー必要
   - **情報**（日次ダイジェストに集約）: Activity自動記録完了、ステータス更新完了
3. 同一商談への重複通知は最後の通知から4時間以内は抑制する
4. 翌日の優先タスクリストを生成する（P1→P2の順、最大10件）
5. `common/output-structure-patterns.md` の日次サマリーテンプレートに従いフォーマットする
6. 配信先: #sales-pipeline Slackチャンネル（日次ダイジェスト）+ 緊急案件は担当者Slack DM
7. **Wait for Explicit Approval**: 日次サマリーの内容を確認する（CP-6）
8. **MANDATORY**: サマリー配信完了を監査ログに記録する

## R2: Next-Day Prep（CONDITIONAL EXECUTE）

**実行条件**: 夜間モード時（S1でModeが「nighttime」の場合）
**スキップ条件**: 日中モード

詳細ルール: `reporting/next-day-prep.md` を参照

1. Google Calendarから翌営業日のミーティング予定を取得する（外部参加者を含む予定のみ）
2. 各ミーティングについて会議準備サマリーを事前生成する:
   - 対応するSalesforce商談情報（ステージ・金額・クロージング日・担当者）
   - 直近のGmailやりとり（最新3件のスニペット）
   - 過去の議事録・Activityログのサマリー
3. 翌日クロージング期限の商談担当者へ事前アラートを準備する
4. **夜間制約**: 顧客向けアクション（メール送信等）は一切実行しない。サマリー生成のみ行う
5. 生成したサマリーをSalesforceのActivityドラフトとして保存する（翌営業日に担当者が確認）
6. **Wait for Explicit Approval**: 翌日準備内容（会議サマリー件数・アラート件数）を確認する（CP-7）
7. **MANDATORY**: 翌日準備完了を監査ログに記録する

## R3: Audit Log Finalization（ALWAYS EXECUTE）

詳細ルール: `reporting/audit-log-finalization.md` を参照

1. 本サイクルで発生した全イベントの監査ログを確定する:
   - サイクルID（UUID）・開始時刻・終了時刻・動作モード
   - 各ステージの実行結果（ステージ名・開始/終了時刻・処理件数・エラー）
   - 全アクション履歴（アクションID・対象商談ID・アクション種別・実行者/承認者・結果・理由）
   - 承認記録（承認リクエスト送信時刻・承認者・承認時刻・承認結果）
   - ロールバック記録（発生有無・対象アクション・復元結果）
   - ループカウント・繰り延べ件数・タイムアウト有無
2. タイムスタンプはISO 8601形式（例: `2026-04-05T09:00:00+09:00`）を使用する
3. 監査ログはappend-onlyで書き込む。既存ログの上書き・削除は禁止
4. ログの完全性チェック: 記録件数と実行件数の一致を確認する
5. サイクル状態ファイルをアーカイブし、次サイクル用に初期化する（繰り延べリストは引き継ぐ）
6. **MANDATORY**: 監査ログ確定完了の最終エントリを追記する

---

# BUILD-TIME: PACKAGING（運用フローとは独立）

**目的**: 検証済みポリシーをClaude Codeプラグイン形式にパッケージング
**注意**: このフェーズはポリシー初期構築/更新時のみ実行する。巡回サイクルのランタイムフローには含まれない

---

## P1: Plugin Structure Generation（ALWAYS EXECUTE）

1. 全ポリシーファイルの確定を確認する
2. plugin.json と marketplace.json を生成する
3. エージェント定義・スキルエントリポイント・コマンドを生成する
4. `rule-details/` のルール詳細ファイルをプラグイン構造にコピーする
5. **Wait for Explicit Approval**: プラグイン構造（ファイル一覧・参照整合性チェック結果）を提示する（CP-8）

## P2: Automated Validation（ALWAYS EXECUTE）

1. 3層テストを実行する:
   - **Layer 1 — 構造テスト**: 全32ファイルの存在確認、相互参照の解決（リンク切れなし）、Markdown妥当性（見出し階層の飛びなし）、フローパスの完全性（デッドエンドなし）
   - **Layer 2 — コンテンツテスト**: Dim12（ドメイン特化率 >= 40%）、Dim13（GOOD/BADパターン >= 2/ファイル）、Dim14（テンプレート完備: D2/D3/R1/R2/R3/A1の6ファイル）、Dim15（5 Pitfalls引用率 >= 50%）、ループ制御設定値（5ループ/45分/30分/2時間）、承認境界一貫性、多層判定必須化
   - **Layer 3 — スモークテスト**: 11フローパターンの到達可能性（日中巡回/夜間準備/承認承認/承認却下/承認タイムアウト/ループ上限/45分タイムアウト/ロールバック/データソース障害/セーフモード/検出なし）
2. テスト失敗時: P1へ戻り最大2ループまで修正する（別カウンター）
3. **Wait for Explicit Approval**: 検証レポート（3層テスト結果・PASS/FAIL）を提示する

---

## Key Principles

- **巡回サイクル型ループ**: 残件が0件またはループ上限（5回）・タイムアウト（45分）に達するまでASSESSMENT→DISPATCHを繰り返す
- **2軸Hybrid自律制御**: 時間帯（日中/夜間）とアクション種別（自動実行/承認制）の2軸でモードを動的に切り替える
- **承認制による安全弁**: 高リスクアクションは必ず人間の承認を経てから実行する。境界曖昧は承認制に分類する（Pitfall 2対策）
- **多層判定による誤検出防止**: 停滞判定はSalesforceデータのみでなく、Gmail/Calendar/Slackのアクティビティを加味する（Pitfall 1・Pitfall 5対策）
- **通知の階層化**: 緊急/重要/情報の3層で通知を分類し、同一商談の重複通知を4時間抑制する（Pitfall 3対策）
- **Salesforce APIガバナーリミット対策**: 使用率80%でセーフモード、95%で読み取り専用モードに自動移行する（Pitfall 4対策）
- **データソース間矛盾の保守的処理**: 矛盾フラグ付き案件は自動判定を禁止し、人間判断を促す（Pitfall 5対策）
- **ロールバック保証**: 全自動実行アクションの事前状態を保存し、異常時は即時ロールバックする
- **監査証跡の完全性**: 全ステージの全イベントをISO 8601タイムスタンプで監査ログに記録する。append-onlyで上書き禁止
- **夜間制約の遵守**: 夜間モード中は顧客向けアクション（メール送信等）を一切実行しない。翌営業日8:00まで保留する
- **NO EMERGENT BEHAVIOR**: 全チェックポイントで標準化された2-option完了メッセージを使用する

## MANDATORY: Completion Message Format

すべてのチェックポイントで以下のフォーマットを使用すること:

```
### REVIEW REQUIRED
[生成内容のサマリー]

### WHAT'S NEXT
**A) Request Changes** — フィードバックに基づいて修正する
**B) Continue to [次のステージ]** — 続行する
```

## Prompts Logging Requirements
- **MANDATORY**: 全インタラクションをISO 8601タイムスタンプ付きで監査ログに記録する
- **MANDATORY**: 完全な生入力をキャプチャする（要約は禁止）
- **CRITICAL**: 監査ログへの追記のみ許可する。上書きは絶対禁止

## Repair Judgment Tree

```
FAIL / Gap detected
├── Structural（ファイル欠損・参照切れ・フローブレーク）
│   → core-workflow.md 再生成
├── Content（ドメイン特化率低・例不足・テンプレート不足）
│   → Phase Rules 再生成 または Common Rules 再生成
├── Design（フェーズ構造不適・ステージ分類誤り・ループ設計不備）
│   → Workflow Architecture 再設計
└── Criteria（品質次元定義不備）
    → Quality Mechanisms 再設計
```

### Domain固有リペアシナリオ

| 症状 | Pitfall起因 | 分類 | 戻り先 |
|------|-----------|------|--------|
| 停滞商談の誤検出多発 | Pitfall 1 / Pitfall 5 | Content | change-detection.md 再生成（多層判定ロジック修正） |
| 高リスクアクションが自動実行されている | Pitfall 2 | Content | action-authority.md + auto-execute.md 修正 |
| 担当者が通知を無視・対応率低下 | Pitfall 3 | Content | daily-summary-generation.md 修正（通知集約ルール強化） |
| Salesforce API制限でサイクル中断 | Pitfall 4 | Design | schedule-check.md + loop-control.md 修正（セーフモード閾値見直し） |
| ソース間矛盾による誤判定多発 | Pitfall 5 | Content | data-collection.md 修正（矛盾検出パターン拡充） |
| ロールバック失敗によるデータ不整合 | Pitfall 2 | Content | rollback-check.md 修正（ロールバックAPI呼び出し修正） |

## Loop Control

- **Max loops / cycle**: 5（1サイクルで対応しきれない件数は次サイクルへ繰り延べ）
- **Cycle timeout**: 45分（1時間サイクルに対して15分バッファを確保）
- **Approval timeout（リマインダー）**: 30分（担当者の対応漏れ防止）
- **Approval timeout（繰り延べ）**: 2時間（承認待ちでエージェントをブロックしない）
- **新規検出の扱い**: 次サイクル繰り延べ（現サイクル中の重複通知・対応件数爆発を防止）
- **Max retries（修復ループ）**: 3（コンテキスト爆発防止）
- **Same-target repair limit**: 2（同一対象への2回連続失敗は問題分類誤りを示唆）
- **P2→P1 loop**: 最大2回（別カウンター）

## Generated Output Directory Structure

```text
sales-pipeline-agent/
├── .claude-plugin/
│   └── plugin.json
├── agents/                                (orchestrator, data-integrator)
├── commands/                              (start-cycle, resume-cycle, validate-policy)
└── skills/
    └── pipeline/
        ├── SKILL.md
        ├── core-workflow.md
        ├── standards.md
        └── rule-details/
            ├── common/                    （12ファイル）
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
            │   ├── action-authority.md
            │   └── data-source-config.md
            ├── surveillance/              （4ファイル）
            │   ├── schedule-check.md
            │   ├── source-connection.md
            │   ├── data-collection.md
            │   └── change-detection.md
            ├── assessment/                （4ファイル）
            │   ├── deal-health-scoring.md
            │   ├── action-identification.md
            │   ├── priority-ranking.md
            │   └── authority-classification.md
            ├── dispatch/                  （5ファイル）
            │   ├── auto-execute.md
            │   ├── approval-request.md
            │   ├── approval-processing.md
            │   ├── loop-control.md
            │   └── rollback-check.md
            ├── reporting/                 （3ファイル）
            │   ├── daily-summary-generation.md
            │   ├── next-day-prep.md
            │   └── audit-log-finalization.md
            └── packaging/                 （2ファイル）
                ├── plugin-structure-generation.md
                └── automated-validation.md
```
