# S1: Schedule Check — Phase Rule

## Purpose

巡回サイクルの開始タイミングを判定する。JST時間帯（日中8:00-20:00 / 夜間20:00-8:00）に基づくモード切り替え、前回サイクルの完了確認と繰り延べリストの引き継ぎ、Salesforce APIガバナーリミット使用量の確認を行い、以降のフローを制御する。

**Classification**: ALWAYS EXECUTE（全巡回サイクルで必須）
**Approval Gate**: あり（CP-1: 時間帯モード確定・API使用率正常確認）
**参照元**: `workflow-architecture.md` S1 セクション

---

## Prerequisites

- 巡回スケジュールトリガーが発火していること（1時間サイクルまたは手動トリガー）
- サイクル状態ファイルへの読み書き権限があること
- Salesforce接続情報（APIキー・ガバナーリミット取得エンドポイント）が設定済みであること
- 日本祝日カレンダー参照ファイル（`config/japan-holidays.json`）が存在すること

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: 常に実行（巡回サイクル開始時の最初のステージ）
- Skip IF: なし（スキップ不可）

---

## Execution Steps

### Step 1: 現在時刻の取得と時間帯判定

現在時刻をJSTで取得し、以下のルールで時間帯モードを判定する。

| 条件 | モード | 後続フロー |
|------|--------|-----------|
| 平日（月〜金）かつ 8:00 <= 現在時刻 < 20:00 | 日中モード | S2（Source Connection）へ続行 |
| 上記以外（20:00-8:00、土日、祝日） | 夜間モード | R2（Next-Day Prep）へジャンプ |

**祝日判定**:
1. `config/japan-holidays.json` から当日が日本祝日かを確認
2. 会社カレンダー設定（`config/company-calendar.json`）に独自休日定義がある場合は追加適用
3. 祝日かつ8:00-20:00の場合でも **夜間モード** として扱う（顧客向けアクション抑制のため）

**時刻境界の扱い**:
- 8:00 ちょうど: 日中モード
- 20:00 ちょうど: 夜間モード
- タイムゾーン変換は常にJST（UTC+9）を使用。サーバーのローカル時刻には依存しない

### Step 2: 前回サイクルの完了確認と繰り延べリスト引き継ぎ

サイクル状態ファイル（`state/current-cycle.md`）を参照し、前回サイクルの状態を確認する。

**前回サイクル状態の判定**:

| 前回状態 | 対処 |
|---------|------|
| `completed` | 繰り延べリストを読み込み、現サイクルの初期キューに追加 |
| `timeout` (45分タイムアウトで強制終了) | 繰り延べリストを引き継ぐ。タイムアウトフラグを現サイクルのログに記録 |
| `error` (異常終了) | 管理者Slack通知を送信。繰り延べリストを保護した上で引き継ぐ |
| 状態ファイルなし | 初回起動として扱い、繰り延べリストを空で初期化 |

**繰り延べリスト構造**:
```yaml
deferred_actions:
  - action_id: "ACT-20260405-001"
    type: "approval_timeout"
    target_opportunity_id: "006xxxxxxxxxxxx"
    deferred_at: "2026-04-05T10:30:00+09:00"
    reason: "approval_not_received_2h"
  - action_id: "ACT-20260405-002"
    type: "loop_limit_reached"
    target_opportunity_id: "006yyyyyyyyyyyyy"
    deferred_at: "2026-04-05T10:45:00+09:00"
    reason: "max_loops_5_reached"
```

### Step 3: Salesforce APIガバナーリミット使用量の確認

Salesforce `limits` APIエンドポイントを呼び出し、当日のAPI使用量を取得する。

```
GET /services/data/v58.0/limits
```

**使用量判定と対処**:

| API使用率 | モード | 対処内容 |
|-----------|--------|---------|
| < 80% | 通常モード | 1時間サイクルで巡回続行 |
| 80% 以上 95% 未満 | セーフモード | 巡回間隔を2時間に延長。管理者Slack通知（`#sales-pipeline-ops`）送信 |
| 95% 以上 | 読み取り専用モード | DML操作（Salesforceへの書き込み）を全てブロック。巡回・読み取りのみ続行 |

**セーフモード中の承認済みアクション**:
- Salesforceへの書き込みを保留し、次サイクルへ繰り延べ
- 保留アクションを繰り延べリストに追記
- 担当者への通知（Slack DM）のみ実行（書き込みを伴わない）

**読み取り専用モード中の制約**:
- D1（Auto-Execute）でのSalesforce更新: 全てスキップし繰り延べ
- D3（Approval Processing）でのSalesforce更新: 全てスキップし繰り延べ
- Gmail/Slack通知の送信: 制約なし（Salesforce外のアクションは実行可能）

**ガバナーリミット対象の主要リソース**:
- `DailyApiRequests`: 日次API呼び出し総数（最重要）
- `DailyBulkApiBatches`: Bulk API 2.0バッチ数
- `DailyDMLRows`: DML操作行数

### Step 4: 新サイクル状態ファイルの初期化

現サイクルのIDを発行し、状態ファイルを初期化する。

```markdown
# Sales Pipeline Agent — Cycle State

## Current Cycle
- **Cycle ID**: [新規UUID]
- **Start Time**: [ISO 8601 JST]
- **Mode**: [daytime / nighttime]
- **API Usage**: Salesforce: [X]%, Gmail: OK, Calendar: OK, Slack: OK
- **Safe Mode**: [true / false]
- **Read Only Mode**: [true / false]
- **Loop Count**: 0 / Max: 5
- **Elapsed**: 0分 / Timeout: 45分
- **Deferred Actions Carried Over**: [件数]

## Processing History
- [timestamp] S1: Mode=[daytime/nighttime], API usage=Salesforce [X]%
  [SafeMode/ReadOnlyMode の場合は追記]
```

---

## Output Artifacts

S1 が生成・更新するアーティファクト:

1. **サイクル状態ファイル**（`state/current-cycle.md`） — 時間帯モード・API状態・繰り延べリストを記録。S2以降で参照
2. **繰り延べリスト**（`state/deferred-actions.yaml`） — 前回サイクルから引き継いだアクション一覧
3. **モード判定結果**（日中 / 夜間） — S2またはR2への分岐制御に使用
4. **APIモード判定結果**（通常 / セーフ / 読み取り専用） — D1・D3の実行制約に使用

---

## Completion Message

S1 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — S1: Schedule Check 完了

- 時間帯モード: [日中モード（8:00-20:00）/ 夜間モード（20:00-8:00）]
- Salesforce API使用率: [X]%（[通常 / セーフモード / 読み取り専用]）
- 繰り延べアクション引き継ぎ: [件数]件
- 前回サイクル状態: [completed / timeout / error / 初回]

次のステップ:
- 日中モードの場合: S2（Source Connection）へ続行
- 夜間モードの場合: R2（Next-Day Prep）へジャンプ

### WHAT'S NEXT
**A) 修正依頼** — モード判定や設定値を修正する
**B) [S2 Source Connection / R2 Next-Day Prep]へ続行** — 判定結果に基づいて続行する
```

---

## Error Handling

### エラー 1: 繰り延べリストのリセット漏れ（Pitfall 4対策）

**症状**: 前回サイクルが45分タイムアウトで強制終了していたにもかかわらず、繰り延べリストをリセットし未対応アクションが消失する。
**原因**: 状態ファイルのタイムアウトフラグ確認ロジックが欠如。
**対処**: Step 2でサイクル状態を必ず確認し、`timeout` または `error` 状態の場合は繰り延べリストをリセットせず引き継ぐ。

**GOOD例**:
```
前回サイクル状態: timeout（経過45分で強制終了）
→ 繰り延べリストに残件3件を確認
→ 現サイクルの初期キューに3件を引き継ぎ
→ 状態ファイルに「deferred_actions_carried_over: 3」を記録
→ 未対応アクションの消失なし（Pitfall 4対策）
```

**BAD例**:
```
前回サイクル状態: timeout（経過45分で強制終了）
→ 繰り延べリストを確認せずリセット
→ 残件3件が消失
→ 停滞商談のフォローアップが実施されず、翌週の商談レビューで発覚
（繰り延べリスト未確認による未対応アクション消失）
```

### エラー 2: APIガバナーリミット未確認による巡回中断（Pitfall 4対策）

**症状**: Salesforce API使用率が95%に達した状態で書き込み系APIを呼び出し、ガバナーリミットエラーで巡回が中断する。
**原因**: Step 3のAPI使用量確認をスキップ、またはリミット確認APIの呼び出し失敗を無視。
**対処**: API使用量取得に失敗した場合はセーフモードとして扱い、書き込み系操作を保守的に制限する。

**GOOD例**:
```
Salesforce API使用率: 83%
→ セーフモードに移行
→ 巡回間隔を2時間に延長
→ 管理者Slack通知: 「API使用率83%: セーフモードに移行しました」
→ ガバナーリミット到達を回避し、巡回を継続（Pitfall 4対策）
```

**BAD例**:
```
Salesforce API使用率確認をスキップ（接続タイムアウト）
→ 通常モードで巡回継続
→ D1での自動実行中にAPI使用率が100%に到達
→ DMLエラーで巡回が中断、未処理アクションが残存
（API使用量未確認による巡回中断）
```

### エラー 3: タイムゾーン誤設定による時間帯誤判定

**症状**: サーバーがUTCで稼働しているため、JST 20:00（UTC 11:00）を日中モードと誤判定し、顧客向けメールを夜間に送信する。
**原因**: 時刻取得時にタイムゾーン変換を省略。
**対処**: 時刻は常に `Asia/Tokyo` タイムゾーンで取得・比較する。サーバーのシステムロケールに依存しない。

**GOOD例**:
```
現在時刻 (UTC): 2026-04-05T11:00:00Z
→ JST変換: 2026-04-05T20:00:00+09:00
→ 20:00 ちょうど = 夜間モード
→ R2（Next-Day Prep）へジャンプ
→ 顧客向けアクション抑制（夜間制約適用）
```

**BAD例**:
```
現在時刻 (UTC): 2026-04-05T11:00:00Z
→ UTCのまま 11:00 と判断
→ 日中モードと誤判定
→ 承認済み顧客メールをJST 20:00に送信
→ 顧客が夜間にメールを受信し不快感
```

### エラー 4: 祝日カレンダー未参照（ドメイン固有）

**症状**: 日本の祝日（例: ゴールデンウィーク）に日中モードと判定され、顧客向けアクションが実行される。
**原因**: `config/japan-holidays.json` が未更新、またはStep 1の祝日チェックが省略されている。
**対処**: 祝日カレンダーファイルが見つからない場合は保守的に夜間モードとして扱う。ファイルが古い（1年以上未更新）場合は管理者に更新アラートを送信する。

**GOOD例**:
```
当日: 2026-05-04（みどりの日）
→ japan-holidays.json に 2026-05-04 が祝日として登録済み
→ 8:00-20:00 であっても夜間モードとして判定
→ 顧客向けアクション抑制
```

**BAD例**:
```
当日: 2026-05-04（みどりの日）
→ japan-holidays.json が2025年分までしか登録されていない
→ 祝日チェックで「祝日でない」と判定
→ 日中モードで巡回開始、顧客へのフォローアップメールを送信
→ 祝日に顧客メールが届き信頼性低下
```
