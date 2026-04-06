# T3: Priority Assessment — Phase Rule

## Purpose

T1・T2 の結果をもとに優先度スコアを計算し、SLA を確定する。SLA 残時間の監視とSLA 超過警告の発動、営業時間ベースの実効残時間計算、チェックポイント CP-1 でのトリアージサマリー提示を行う。

**Classification**: ALWAYS EXECUTE（全問い合わせで必須）
**Approval Gate**: あり（CP-1: トリアージ結果の確認）
**参照元**: `core-workflow.md` T3 セクション

---

## Prerequisites

- T1 が完了しており、顧客セグメント・プラン・感情シグナル・反復回数が確定していること
- T2 が完了しており、プライマリカテゴリが確定していること（`pending_manual` の場合は CP-1 で同時確認）
- 営業日カレンダー（祝日情報）が参照可能であること

---

## Execution Steps

### Step 1: 優先度スコア計算

以下の計算式で優先度スコアを算出する。

```
優先度スコア = セグメント重み × カテゴリ重み × 緊急度指標 × 反復係数
```

**セグメント重み**:

| セグメント | 重み |
|----------|-----|
| VIP | 3.0 |
| Complaint-in-progress | 2.5 |
| Enterprise（プラン） | 2.0 |
| Pro（プラン） | 1.5 |
| General / Free | 1.0 |

※ セグメントとプランが重複する場合（VIP かつ Enterprise など）は高い方の値を採用する。

**カテゴリ重み**:

| カテゴリ | 重み |
|---------|-----|
| Security | 5.0 |
| Bug | 3.0 |
| Billing | 2.5 |
| Feature | 1.5 |
| Account | 1.5 |

**緊急度指標**:

| 条件 | 指標値 |
|-----|-------|
| 件名または本文に「緊急」「至急」「URGENT」等 | 1.5 |
| 感情シグナル: `angry` | 1.4 |
| 感情シグナル: `frustrated` | 1.2 |
| 感情シグナル: `confused` または `normal` | 1.0 |

**反復係数（v2）**:

| 反復回数（repeat_count） | 係数 |
|----------------------|-----|
| 0〜1 | 1.0 |
| 2 以上 | 1.5 |

**計算例**:
```
顧客: Enterprise / 感情: frustrated / カテゴリ: Bug / repeat_count: 2
スコア = 2.0 × 3.0 × 1.2 × 1.5 = 10.8 → P1（Critical）
```

### Step 2: 優先度レベル確定

優先度スコアをもとに P1〜P4 を確定する。

| 優先度レベル | スコア範囲 | 意味 |
|-----------|---------|-----|
| P1（Critical） | >= 8.0 | 最優先。即時対応が必要 |
| P2（High） | 4.0〜7.9 | 高優先。優先キューに投入 |
| P3（Medium） | 2.0〜3.9 | 通常処理 |
| P4（Low） | < 2.0 | 低優先。時間的余裕あり |

**Security カテゴリの強制 P1 ルール**: カテゴリが Security の場合、スコアに関わらず優先度を **P1** に強制設定する。

### Step 3: SLA テーブルと FRT 確定

**SLA テーブル（FRT: First Response Time）**:

| 条件 | FRT | 備考 |
|-----|-----|------|
| Security インシデント | **1 時間** | プラン・セグメントに関わらず適用 |
| Enterprise プラン | **2 時間** | Security 以外のカテゴリ |
| Pro プラン | **8 時間** | Security 以外のカテゴリ |
| Free プラン / 顧客不明 | **24 時間** | Security 以外のカテゴリ |

**優先度と SLA の関係**: SLA は顧客プランに基づき確定する。P1 だからといって FRT が自動短縮されるわけではない（ただし Security カテゴリは例外として 1h FRT）。P1 はキュー内の処理順序を上げる意味を持つ。

### Step 4: 営業時間ベースの実効 SLA 残時間計算

FRT は営業時間内の経過時間で計算する。

**営業時間定義**:
- タイムゾーン: JST（UTC+9）
- 営業日: 月曜日〜金曜日
- 営業時間: 09:00〜18:00 JST
- 休業日: 土日・祝日（祝日カレンダーを参照）

**実効残時間計算ロジック**:

1. 受信日時（`ticket.email.received_at`）が営業時間内かを確認する
2. 営業時間外の受信: FRT カウントは翌営業日 09:00 から開始する
3. FRT 内に営業時間外をまたぐ場合: 営業時間外の時間を FRT カウントから除外する

**計算例**:
```
受信: 金曜日 17:30 JST / プラン: Pro（FRT: 8h）
営業時間残: 30分（17:30〜18:00）
月曜日 09:00 からカウント再開
実効 SLA 期限: 月曜日 09:00 + (8h - 0.5h) = 月曜日 16:30 JST
```

**Security インシデントの例外**: Security カテゴリの 1h FRT は営業時間外でも実時間でカウントする（24時間対応）。

### Step 5: SLA 警告・超過フラグ管理（v2）

**SLA 警告フラグ（Warning）**:
- 条件: SLA 期限まで残り 15 分以下
- アクション: チケットに `sla_status: warning` を付与し、処理キューの最上位に移動する
- 通知: 担当エージェントにアラートを送信する

**SLA 超過フラグ（Exceeded）**:
- 条件: SLA 期限を超過している
- アクション: チケットに `sla_status: exceeded` を付与する
- 通知: 管理者（マネージャー）へ自動通知メールを送信する
- 処理ショートカット: Q2（Content Review）を簡易チェックに切り替える（Q1 PII Scan は省略不可）

**SLA ステータス値域**:

| ステータス | 条件 |
|---------|-----|
| `on_track` | 期限まで 15 分超 |
| `warning` | 期限まで 15 分以下 |
| `exceeded` | 期限超過 |

### Step 6: チケットへの優先度・SLA 情報の記録

```yaml
priority:
  score: "{計算値}"
  level: "{P1 | P2 | P3 | P4}"
  segment_weight: "{値}"
  category_weight: "{値}"
  urgency_indicator: "{値}"
  repeat_factor: "{値}"
  force_p1_reason: "{Security強制P1 | null}"

sla:
  frt_hours: "{1 | 2 | 8 | 24}"
  frt_deadline: "{ISO 8601タイムスタンプ}"
  business_hours_only: "{true | false（Securityはfalse）}"
  sla_status: "{on_track | warning | exceeded}"
  sla_exceeded_at: "{ISO 8601タイムスタンプ | null}"

audit_log:
  - timestamp: "{ISO 8601}"
    stage: "T3"
    action: "priority_assessed"
    details: "priority: {P-level}, SLA: {frt_hours}h, status: {sla_status}"
```

---

## Question Generation（CP-1）

T3 完了後、以下のトリアージサマリーを提示しユーザー確認を求める（CP-1）。

**確認内容**:
- 分類結果（カテゴリ・サブカテゴリ・信頼度）
- 優先度（Pレベル・スコア・算出根拠）
- SLA（FRT 期限・ステータス）
- 特記事項（SLA 超過、分類要確認、感情シグナルなど）

**CP-1 提示テンプレート**:

```
### REVIEW REQUIRED
トリアージが完了しました。内容を確認してください。

**チケット情報**
- Ticket ID: {TKT-xxx}
- 顧客: {メールアドレス} / セグメント: {segment} / プラン: {plan}
- 感情シグナル: {emotion_signal}
- 反復回数: {repeat_count}

**分類結果**
- カテゴリ: {primary_category} / {subcategory}
- 分類信頼度: {score}
- ステータス: {confirmed | ⚠️ pending_manual（手動確認が必要）}

**優先度・SLA**
- 優先度: {P1 | P2 | P3 | P4}（スコア: {score}）
- FRT 期限: {ISO 8601} ({frt_hours}h SLA)
- SLA ステータス: {on_track | ⚠️ warning | 🚨 exceeded}

**特記事項**
{SLA超過の場合: 🚨 SLA超過: 管理者に自動通知済み}
{Security強制P1の場合: ⚠️ Security インシデントのため P1 強制適用}
{分類要確認の場合: ⚠️ 分類信頼度が低いため手動確認が必要}

### WHAT'S NEXT
**A) 修正依頼** — カテゴリ・優先度を修正する
**B) R1（Mode Selection）へ続行** — 対応モード選択へ進む
```

---

## Output Artifacts

T3 が生成するアーティファクト:

1. **優先度情報**（チケットオブジェクトの `priority` フィールドに追記）— R1/R4 で参照
2. **SLA 情報**（チケットオブジェクトの `sla` フィールドに追記）— R1/Q2/Q4 で参照
3. **管理者通知**（`sla_status: exceeded` の場合のみ）
4. **監査ログエントリ** — `stage: T3, action: priority_assessed`

---

## Completion Message

CP-1 は Question Generation セクションの CP-1 提示テンプレートで対応する。

---

## Error Handling

### エラー 1: 営業時間を考慮しない SLA 計算（Pitfall: SLA 違反）

**症状**: 金曜日 17:00 受信の Enterprise チケット（FRT: 2h）のSLA 期限を「金曜日 19:00」と計算し、月曜日朝まで対応しない。  
**原因**: 実効残時間計算で営業時間外を考慮していない。  
**対処**: 必ず `business_hours_only: true`（Security 以外）のロジックで SLA 期限を計算する。実装は Step 4 の計算ロジックに準拠する。

**GOOD例**:
```
受信: 金曜日 17:30 JST / Enterprise / Bug（FRT: 2h）
営業時間残: 30分
実効 SLA 期限: 月曜日 09:00 + 1.5h = 月曜日 10:30 JST
月曜日 10:15 時点で sla_status: warning に移行
```

**BAD例**:
```
受信: 金曜日 17:30 JST / Enterprise / Bug（FRT: 2h）
実効 SLA 期限: 金曜日 19:30 JST（営業時間外を無視）
→ 月曜朝に確認した時点では既に大幅超過
→ SLA 超過ログが残り、顧客との契約違反
```

### エラー 2: Security カテゴリの SLA が 2h・8h・24h で計算される（Pitfall: エスカレーション判定ミス）

**症状**: Enterprise 顧客のセキュリティ報告に対し、プランベースの 2h SLA を適用してしまう。  
**原因**: Security カテゴリの 1h FRT 強制ルールが実装されていない、またはプランベース SLA が先に評価されている。  
**対処**: SLA テーブルの評価順序を「Security インシデント → プランベース」の順に固定する。

**GOOD例**:
```
顧客: Pro プラン / カテゴリ: Security/unauthorized_access
SLA 評価: Security インシデント → FRT: 1h（プランに関わらず）
sla_status: on_track（受信から 45 分経過の場合 warning 手前）
Priority: P1（強制）
```

**BAD例**:
```
顧客: Pro プラン / カテゴリ: Security/unauthorized_access
SLA 評価: Pro プラン → FRT: 8h
Priority: P2（スコアベース）
→ 不正アクセスへの対応が最大 8 時間遅延
→ データ損失リスクが著しく上昇
```

### エラー 3: 反復係数の未適用（Pitfall: エスカレーション判定ミス）

**症状**: `repeat_count: 3` のチケットなのにスコアが低く P3 のまま。  
**原因**: T1 で `repeat_count` が正しく更新されているが、T3 の計算式に反復係数が組み込まれていない。  
**対処**: Step 1 の計算式で `repeat_count >= 2` の場合は必ず × 1.5 を乗じる。T1 から引き継がれた `repeat_count` 値を T3 が参照していることを確認する。

**GOOD例**:
```
顧客: General / カテゴリ: Account / 感情: frustrated / repeat_count: 3
スコア = 1.0 × 1.5 × 1.2 × 1.5 = 2.7 → P3
（反復係数なしなら 1.8 → P4 で埋もれる）
→ キューで適切に優先処理される
```

**BAD例**:
```
顧客: General / カテゴリ: Account / 感情: frustrated / repeat_count: 3
スコア = 1.0 × 1.5 × 1.2 × 1.0 = 1.8（反復係数未適用）→ P4
→ 3 回繰り返している顧客が最低優先度に置かれ、チャーンリスクが高まる
```

### エラー 4: SLA 超過後の Q1 スキップ（Pitfall: PII 漏洩）

**症状**: SLA 超過で急いで回答を送信するため Q1 PII Scan を省略する。  
**原因**: SLA 超過ショートカットの解釈誤り（Q2 を簡易化するが Q1 は省略不可）。  
**対処**: `sla_status: exceeded` 時のショートカットは Q2（Content Review）の簡易化のみ。Q1（PII Scan）は `core-workflow.md` の MANDATORY ルールにより、いかなる状況でも省略不可である。T3 のトリアージサマリー（CP-1）で「Q1 は省略不可」を明記して確認を取る。
