# Q4: Dispatch（ディスパッチ）

## Purpose

全品質チェック（Q1〜Q3）を通過した出力の最終確認（CP-5）を行い、モード別の出力先へ送信・引き継ぎ・レポート出力を実行する。監査ログへの完全な記録、SLA達成/未達の記録と超過時の管理者通知、CSATアンケートの自動付与を担う。Q3から強制エスカレーションフラグが渡された場合はエスカレーションモードで処理を強制完結する。

**Classification**: ALWAYS EXECUTE

**参照元**: `core-workflow.md` Q4、`common/feedback-loop-and-retention.md`

---

## Prerequisites

- Q3（Tone Calibration）が完了し、トーンキャリブレーション済みドラフトが渡されていること
- Q1のPIIスキャンログ、Q2のコンテンツレビューログ、Q3のトーンキャリブレーションログが参照可能であること
- 処理モード（`immediate` / `escalation` / `analysis`）が確定していること
- `FORCED_ESCALATION` フラグの有無が確認可能であること
- SLA残時間・SLA状態（`OK` / `Warning` / `Exceeded`）が記録されていること

---

## Execution Steps

### Step 1: CP-5 送信前最終確認チェックリスト

以下の全項目が `PASS` であることを確認する。1件でも `FAIL` がある場合は送信を停止する。

```
CP-5 最終確認チェックリスト:
□ PII: CLEAR
  → Q1ログで PASS または MASKED が記録されていること
  → FAIL条件: Q1が未実行、またはスキャン結果が ERROR

□ 正確性: VERIFIED
  → Q2ログで PASS または PASS_WITH_WARNINGS が記録されていること
  → FAIL条件: Q2結果が ESCALATE_REQUIRED（別途エスカレーション処理）
  → FAIL条件: Q2が未実行

□ トーン: CALIBRATED
  → Q3ログで PASS または REPAIRED が記録されていること
  → 強制エスカレーションフラグあり → エスカレーションモードに切り替え
  → FAIL条件: Q3が未実行

□ KB引用: PRESENT（即時回答モードのみ必須）
  → ドラフト内に `([KB記事タイトル]: {URL})` 形式の引用が1件以上あること
  → 即時回答モード以外: チェック不要
  → FAIL条件: 即時回答モードでKB引用が0件
```

**強制エスカレーションフラグ処理**:
- `FORCED_ESCALATION` フラグが立っている場合、モードを強制的に `escalation` に変更
- 即時回答ドラフトを廃棄し、R4が生成した中間応答を使用
- KB引用チェックをスキップ（エスカレーションモードのため）

### Step 2: モード別出力処理

CP-5を全項目パスした後、モードに応じた出力を実行する。

#### 即時回答モード

1. 顧客宛メールを送信
   - 宛先: T1で取得した顧客メールアドレス（Q1マスキング後の値ではなく、元の宛先フィールドを使用）
   - 件名: `Re: {元の件名}` または適切な件名
   - 本文: Q3通過済みドラフト
   - CSAT依頼: Step 4の設定に従い付与
2. チケット状態を `Resolved` に更新（チケット管理システム）
3. `Q4 Dispatched: immediate_response` をログ記録

#### エスカレーションモード

1. 顧客宛中間応答メールを送信
   - 内容: 「担当者に引き継ぎます」テンプレート（Q1〜Q3通過済み）
   - CSAT依頼: エスカレーション完了後に付与（即時付与しない）
2. オペレーターチームへ引き継ぎサマリーを送付
   - 送付先: R4で選定したカテゴリ別担当チーム
   - 内容: 引き継ぎサマリー（顧客情報マスキング済み + Q1通過済み）
3. チケット状態を `Escalated` に更新
4. `Q4 Dispatched: escalation` をログ記録
5. F1（Case Tracking）へ引き継ぎフラグを立てる

#### 分析モード（R5）

1. 管理者宛にレポートを送付
   - 送付先: 管理者メールアドレスまたはレポートシステム
   - 内容: 傾向分析レポート（Q1〜Q3通過済み・PII除去済み）
2. チケット状態は変更しない（分析は個別チケット処理ではない）
3. `Q4 Dispatched: analysis_report` をログ記録

### Step 3: SLA記録と管理者通知

```
SLA記録処理:
  SLA状態 == OK:
    → 「SLA達成」を監査ログに記録
    → 処理完了

  SLA状態 == Warning（残15分未満で処理完了した場合）:
    → 「SLA Warning内で完了」を監査ログに記録
    → 管理者通知: 「SLA Warning案件完了（TKT-XXXXXXXX-NNN）」

  SLA状態 == Exceeded:
    → 「SLA超過」フラグを監査ログに記録
    → 管理者への自動通知を送信:
```

**管理者通知テンプレート（SLA超過）**:
```
[SLA超過通知]
チケットID: {TKT-XXXXXXXX-NNN}
顧客セグメント: {VIP/General/Complaint-in-progress}
カテゴリ: {カテゴリ/サブカテゴリ}
SLA目標: {N}時間（{プラン}）
実際の応答時間: {実際の時間}
超過時間: {超過分}分
モード: {immediate/escalation/analysis}
処理担当: {エージェントID}
```

### Step 4: CSATアンケート付与

即時回答モードの場合、以下の設定に従いCSATアンケートを付与する。

```
CSAT付与ルール:
  即時回答モード: 回答メール末尾に付与（デフォルト: ON）
  エスカレーションモード: エスカレーション解決後に付与（F1から制御）
  分析モード: 付与しない（管理者レポートのため）

  付与テンプレート:
  ---
  本日のサポートはお役に立てましたでしょうか？
  1分でお答えいただけるアンケートにご協力ください。
  [満足度を回答する（1〜5）]: {CSAT_SURVEY_URL}
  ---
```

### Step 5: 監査ログ記録

すべての処理完了後、以下のフォーマットで監査ログに記録する。

```
[Q4 DISPATCH AUDIT LOG]
- Timestamp: {ISO 8601}
- Ticket ID: {TKT-XXXXXXXX-NNN}
- Customer Segment: {VIP | General | Complaint-in-progress}
- Category: {カテゴリ/サブカテゴリ}
- Priority: {P1 | P2 | P3 | P4}
- Mode: {immediate | escalation | analysis}
- Processing Stages:
  - T1-T3: {timestamp range}
  - R1-R{n}: {timestamp range}
  - Q1: {timestamp} / Result: {PASS | MASKED}
  - Q2: {timestamp} / Result: {PASS | PASS_WITH_WARNINGS | SIMPLIFIED}
  - Q3: {timestamp} / Result: {PASS | REPAIRED | FORCED_ESCALATION}
  - Q4: {timestamp} / Result: DISPATCHED
- Total Processing Time: {分}分
- SLA Target: {N}時間
- SLA Status: {Met | Warning | Exceeded}
  - Exceeded Details: {超過時間}分超過（管理者通知済み）
- CP-5 Checks: PII={PASS}, Accuracy={PASS}, Tone={PASS}, KB={PASS|N/A}
- Dispatch Target: {customer_email | operator_team | manager_report}
- CSAT Attached: {YES | NO | PENDING（escalation後）}
- Follow-up Required: {YES（F1へ）| NO}
```

---

## Question Generation

CP-5チェックリストで `FAIL` が発生した場合、以下の確認を生成する（通常は自動処理）。

```
### CP-5 最終確認 — 確認項目あり
以下の項目がFAILしています:

- {FAIL項目}: {理由}

A) 問題を修正して再チェック — {修正先のフェーズ}に戻ります
B) エスカレーションとして処理を継続 — R4エスカレーションモードで強制完了
```

---

## Output Artifacts

- **送信済み顧客メール**（即時回答・エスカレーション中間応答）
- **引き継ぎサマリー**（エスカレーションモード）
- **分析レポート**（分析モード）
- **完全な監査ログエントリ**: ISO 8601タイムスタンプ付き
- **SLA超過通知**（SLA超過時のみ）
- **F1フォローアップフラグ**（エスカレーション/保留時）

---

## Completion Message

```
### REVIEW REQUIRED
[送信完了]
- モード: {immediate / escalation / analysis}
- CP-5: 全項目 PASS
- 送信先: {宛先}
- SLA: {達成 / Warning内完了 / 超過（管理者通知済み）}
- CSAT: {付与済み / エスカレーション後付与 / 対象外}
- フォローアップ: {F1へ引き継ぎ / 不要}

### WHAT'S NEXT
**A) 修正依頼** — 送信内容に問題がある場合、内容を指定してください（送信取り消しはシステム制約を確認）
**B) フォローアップ（F1）へ続行** — エスカレーション/保留案件の追跡に進む
```

---

## Error Handling

### エラー1: CP-5チェックでPII FAILが発生

**シナリオ**: Q1ログが存在しない（スキップされた可能性）

**原因候補**:
- Pitfall 2（PII漏洩）の最終防衛ライン突破リスク
- フロー制御の不具合でQ1が実行されなかった

**対応**:
1. 送信を即座に停止
2. Q1を強制実行してから再度CP-5チェック
3. `Q1_BYPASS_DETECTED` ログを記録し、インシデントとして管理者に通知
4. 「Q1はSLA状態問わず省略不可」のルールを再確認

### エラー2: 送信システムへの接続エラー

**シナリオ**: メール送信APIがタイムアウトまたはエラーを返した

**対応**:
1. 3回まで自動リトライ（指数バックオフ: 10秒→30秒→90秒）
2. 3回失敗した場合、`DISPATCH_FAILED` ログを記録し管理者にアラート
3. ドラフトをキューに保存（データ損失防止）
4. SLA超過になる場合は超過フラグを立てて管理者通知
5. チケット状態を `Pending_Dispatch` として追跡

### エラー3: KB引用チェック FAIL（即時回答モード）

**シナリオ**: Q2で `UNVERIFIED_CLAIM` を削除した結果、KB引用が0件になった

**原因候補**:
- Pitfall 1（ハルシネーション）: KB外情報がすべて削除され、引用できる根拠がない

**対応**:
1. R4エスカレーションへの切り替えを実行（KB引用なしでの即時回答は不可）
2. モードを `escalation` に変更、「確認の上ご回答します」テンプレートを使用
3. `KB_CITATION_EMPTY_ESCALATION` ログを記録

### エラー4: 強制エスカレーションでも送信可能なドラフトがない

**シナリオ**: Q3でFORCED_ESCALATIONフラグが立ったが、R4の中間応答ドラフトも存在しない

**対応**:
1. 最低限の中間応答テンプレートを直接生成:
```
{顧客名}様、この度はご連絡いただきありがとうございます。
お客様のご状況を確認し、専任の担当者より改めてご連絡いたします。
```
2. このテンプレートはQ1チェック対象（PIIが含まれないため通常はPASS）
3. `EMERGENCY_TEMPLATE_USED` ログを記録

---

## GOOD / BAD 具体例

### GOOD 例1: 全チェックPASS + SLA達成 + 監査ログ完備

**シナリオ**: Proプラン顧客からの一般機能問い合わせ。SLA: 8時間、実際の応答: 45分。

**GOOD（Q4処理後）**:
```
[Q4 DISPATCH AUDIT LOG]
- Timestamp: 2024-01-15T10:45:23+09:00
- Ticket ID: TKT-20240115-042
- Category: Feature/Navigation
- Priority: P3
- Mode: immediate
- Q1: MASKED（メールアドレス1件）
- Q2: PASS_WITH_WARNINGS（KB記事1件が91日超 → 免責文付与済み）
- Q3: PASS（normal感情 × General）
- Q4: DISPATCHED
- SLA Status: Met（45分 / 8時間目標）
- CSAT Attached: YES
```

**解説**: 全フェーズが適切に処理され、SLA達成。KB鮮度警告も免責文で対応済み。

### GOOD 例2: SLA超過時の管理者通知

**シナリオ**: Enterpriseプラン顧客のBug報告。SLA: 2時間、実際の応答: 2時間15分。

**GOOD（SLA超過処理）**:
- Q4でSLA超過フラグを記録
- 管理者通知を自動送信（超過15分）
- ドラフトのQ1〜Q3品質チェックは完全実施
- 送信は品質チェック完了後に実施（超過してもスキップしない）

### BAD 例1: PII Scanスキップで個人情報含む回答を送信

**BAD（NG行動）**:
SLA残0分のためQ1〜Q3をまとめてスキップし、R3ドラフトをそのままCP-5へ渡す。

**CP-5チェック**:
- PII: **FAIL**（Q1未実行）

**正しい処理**: Q1のFAILを検出した時点で送信停止。Q1を強制実行してから再チェック。Pitfall 2（PII漏洩）の最終防衛ライン。

### BAD 例2: 監査ログの不完全記録

**BAD（NG行動）**:
```
[Q4 LOG]
送信完了: TKT-001
```
タイムスタンプなし、品質チェック結果なし、SLA記録なし。

**GOOD（正しい監査ログ）**:
Step 5のフルフォーマットで全項目を記録。ISO 8601タイムスタンプ必須。監査ログはappend-onlyで上書き禁止。
