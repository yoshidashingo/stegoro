---
name: sales-pipeline
description: B2B SaaS営業パイプライン管理スキル。Salesforce/Gmail/Calendar/Slackの4ソースを定期巡回し商談の停滞・未返信・期限接近を自動検出するHybrid型ステアリングポリシーのエントリポイント。
---

## Activation Trigger

以下のいずれかに該当する場合、このスキルを起動する。

- `/start-cycle` または `/resume-cycle` コマンドが実行された場合
- 営業パイプラインの巡回・分析・アクション実行を依頼された場合
- 「パイプライン確認して」「商談チェックして」「停滞商談を教えて」等のキーワードが含まれる場合
- スケジューラーから定期実行トリガーが届いた場合

## Core Rules

### Rule 1: 多層判定の必須適用（Pitfall 1対策）

商談の停滞・異常を判定する際は、Salesforceの単一ソースに依存せず、Gmail・Slack・Calendarのアクティビティを必ず加味する。

**判定式**:
```
停滞 = Salesforce滞留日数 >= 閾値
       AND Gmail/Slack/Calendar直近アクティビティなし
```

### Rule 2: 権限境界の厳守（Pitfall 2対策）

エージェントは低リスクアクション（内部記録・通知・下書き生成）のみ自動実行する。顧客向け送信・ステージ更新・金額変更は必ず担当者承認を得てから実行する。

**承認制必須**: 顧客向けメール送信・Salesforceステージ更新・金額/クロージング日の変更

### Rule 3: 1商談1アクション原則（Pitfall 3対策）

同一商談に複数の検出結果がある場合、最高優先度の1件のみを主アクションとし、他をコンテキストに付加する。承認リクエストは同一担当者あて最大3件をバッチ化する。

**重複抑制**: 未返信リマインダーは前回送信から4時間以内再送禁止。期限接近通知（P2以下）は48時間以内再送禁止。

### Rule 4: APIガバナーリミット保護（Pitfall 4対策）

Salesforce API日次使用率を毎サイクル開始時に確認し、閾値に応じてモードを切り替える。差分取得（全量取得禁止）とBulk API 2.0優先でAPI消費を最小化する。

**モード**: 通常（<80%）→ セーフ（80-95%、管理者通知）→ 読み取り専用（>=95%、DML停止）

### Rule 5: ソース間矛盾の透明な処理（Pitfall 5対策）

複数ソース間でデータが矛盾する場合、矛盾フラグを付与して自動処理から除外し、人間確認依頼アクションとして登録する。矛盾を隠蔽・無視することは禁止。

## Workflow Summary

| フェーズ | ステージ | 分類 | 概要 |
|---------|---------|------|------|
| SURVEILLANCE | S1 スケジュール確認 | ALWAYS | 日中/夜間モード判定 |
| SURVEILLANCE | S2 ソース接続確認 | ALWAYS | 4ソース並行疎通確認 |
| SURVEILLANCE | S3 差分データ収集 | ALWAYS | 統合ビュー構築 |
| SURVEILLANCE | S4 変更・異常検出 | ALWAYS | 7種ルール適用 |
| ASSESSMENT | A1-A4 | ALWAYS | ヘルススコア算出・アクション生成・優先度付け・権限分類 |
| DISPATCH | D1 自動実行 | ALWAYS | 低リスクアクション即時実行 |
| DISPATCH | D2-D4 承認フロー | ALWAYS | 承認リクエスト送信・処理・ループ制御 |
| DISPATCH | D5 ロールバック検証 | ALWAYS | 実行後の整合性確認 |
| REPORTING | R1 日次サマリー | 日中のみ | サマリー生成・配信 |
| REPORTING | R2 翌日準備 | 夜間のみ | 会議サマリー・翌朝アラート予約 |
| REPORTING | R3 監査ログ確定 | ALWAYS | append-onlyログ記録 |

## Quality Gate

各サイクル終了前に以下の全項目がPASSであることを確認する。

| チェック | 基準 |
|---------|------|
| 権限境界 | 承認制アクションが未承認のまま実行されていないこと |
| 矛盾フラグ | conflict_flags付き商談が全て人間確認依頼登録済みであること |
| API保護 | Salesforce API使用率が95%未満であること |
| 監査ログ | 全アクションがISO 8601タイムスタンプ付きで記録済みであること |

## References

詳細なルール定義は以下のファイルを参照する。

- `../../sales-pipeline-agent-rules/core-workflow.md` — ワークフロー全体定義
- `../../sales-pipeline-agent-rule-details/surveillance/` — S1: schedule-check, S2: source-connection, S3: data-collection, S4: change-detection
- `../../sales-pipeline-agent-rule-details/assessment/` — A1: deal-health-scoring, A2: action-identification, A3: priority-ranking, A4: authority-classification
- `../../sales-pipeline-agent-rule-details/dispatch/` — D1: auto-execute, D2: approval-request, D3: approval-processing, D4: loop-control, D5: rollback-check
- `../../sales-pipeline-agent-rule-details/reporting/` — R1: daily-summary-generation, R2: next-day-prep, R3: audit-log-finalization
- `../../sales-pipeline-agent-rule-details/common/` — action-authority, data-source-config, overconfidence-prevention 他
