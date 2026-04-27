---
name: support
description: SaaS/Webサービス向けカスタマーサポート対応スキル。メール問い合わせのトリアージ・回答生成・品質検査・フォローアップを自動化するHybrid型ステアリングポリシーのエントリポイント。
---

# カスタマーサポートスキル

## Activation Trigger

以下のいずれかに該当する場合、このスキルを起動する。

- 顧客からのメールサポート依頼を処理するよう指示された場合
- `/new-ticket` または `/resume-ticket` コマンドが実行された場合
- 「カスタマーサポート」「問い合わせ処理」「サポートメール対応」等のキーワードが含まれる場合
- チケット管理システムから新規チケットが連携された場合

## Core Rules

このスキルが守る5つの中核ルール。

### Rule 1: KB依存回答（Pitfall 1対策）

すべての回答はナレッジベース（KB）に基づいて生成する。KB信頼度スコアが0.4未満の場合は即時回答を行わず、エスカレーションまたは「確認の上回答します」テンプレートを使用する。KB外情報の生成を発見した場合は即座に削除する。

### Rule 2: PII二重防衛（Pitfall 2対策）

- **一次防衛（T1）**: チケット受付時にPIIフラグを付与し、以降のフェーズで参照可能にする
- **最終防衛（Q1）**: 送信前に必ずPIIスキャンを実行する。SLA超過・モード種別に関わらず省略不可

対象PIIは顧客メールアドレス・電話番号・クレジットカード番号・マイナンバー・住所・IPアドレス・社内スタッフ情報を含む。

### Rule 3: 感情ベースエスカレーション（Pitfall 3対策）

感情シグナル `angry` を検出した場合は二段階防衛で対処する。

- **R1（一次検出）**: `angry` × `Complaint-in-progress` セグメントで即エスカレーション
- **Q3（セーフティネット）**: 全セグメント × `angry` をカバー。R1検出漏れをQ3リペアパスで回収（最大1回）

### Rule 4: SLAアウェアネス

T3で顧客プランに基づくSLA（Free: 48h / Standard: 24h / Pro: 8h / Enterprise: 2h / Security: 1h）を割り当て、全フェーズで残時間を監視する。SLA超過時は管理者に自動通知する。SLA残15分未満ではQ2を簡易チェックモードに切り替えるが、Q1（PIIスキャン）は短縮しない。

### Rule 5: 監査ログ

全フェーズ（T1〜Q4・F1・F2）の処理結果をISO 8601タイムスタンプ付きでappend-onlyログに記録する。SLA達成/超過・PIIマスキング件数・品質チェック結果・送信先をすべて記録する。

## Workflow Summary

### ランタイムフロー（チケット処理ごとに実行）

| フェーズ | ステージ | 分類 | 概要 |
|---------|---------|------|------|
| TRIAGE | T1/T2/T3 | ALWAYS | チケット受付・分類・優先度評価とSLA設定 |
| RESPONSE | R1 | ALWAYS | 3モード（即時回答/エスカレーション/分析）の選択 |
| RESPONSE | R2/R3 | 即時回答時 | KB検索・回答ドラフト生成 |
| RESPONSE | R4 | エスカレーション時 | 引き継ぎサマリー・中間応答生成 |
| RESPONSE | R5 | 分析時 | 傾向分析レポート生成 |
| QUALITY | Q1/Q2/Q3/Q4 | ALWAYS | PIIスキャン・コンテンツレビュー・トーン検査・ディスパッチ |
| FOLLOW-UP | F1 | CONDITIONAL | エスカレーション後・保留後のケーストラッキング |
| FOLLOW-UP | F2 | CONDITIONAL | 週次閾値超過またはCSAT蓄積時のFAQ候補抽出 |

### ビルドタイムフロー（ポリシー構築・更新時のみ実行）

| フェーズ | ステージ | 概要 |
|---------|---------|------|
| PACKAGING | P1 | プラグイン構造生成（plugin.json・agents・skills・commands） |
| PACKAGING | P2 | 3層自動バリデーション（構造・コンテンツ・スモークテスト9フロー） |

## Quality Gate

送信前のCP-5最終確認で以下の全項目がPASSであることを保証する。

| チェック | 基準 |
|---------|------|
| PII: CLEAR | Q1スキャン完了（PASS または MASKED） |
| 正確性: VERIFIED | Q2 KB照合完了（PASS または PASS_WITH_WARNINGS） |
| トーン: CALIBRATED | Q3トーン検査完了（PASS または REPAIRED） |
| KB引用: PRESENT | 即時回答モードで引用1件以上 |

いずれか1件でもFAILの場合は送信を停止する。

## References

詳細なルール定義は以下のファイルを参照する。

```
./core-workflow.md           # ワークフロー全体定義
./standards.md               # 品質基準（15次元）
./rule-details/triage/       # T1: ticket-reception, T2: classification, T3: priority-assessment
./rule-details/response/     # R1: mode-selection, R2: knowledge-retrieval, R3: draft-generation
                             # R4: escalation-handoff, R5: trend-analysis
./rule-details/quality/      # Q1: pii-scan, Q2: content-review, Q3: tone-calibration, Q4: dispatch
./rule-details/follow-up/    # F1: case-tracking, F2: faq-candidate-extraction
./rule-details/packaging/    # P1: plugin-structure-generation, P2: automated-validation
./rule-details/common/       # tone-guidelines, content-validation, overconfidence-prevention 他
```
