# T1: Ticket Reception — Phase Rule

## Purpose

受信メールを解析し、顧客情報を抽出する。PII初期検出・フラグ付与、感情シグナル検出、重複チケット検出を行い、後続ステージが利用するチケットオブジェクトを生成する。

**Classification**: ALWAYS EXECUTE（全問い合わせで必須）
**Approval Gate**: なし（次ステージ T2 へ自動移行）
**参照元**: `core-workflow.md` T1 セクション

---

## Prerequisites

- 受信メールが存在すること（From, Subject, Body 最低限）
- `common/content-validation.md` を読み込み済み（PIIパターン正規表現の参照元）
- `common/terminology.md` を読み込み済み（チケット/トリアージ等用語の参照元）
- チケット管理システムへの書き込み権限があること

---

## Execution Steps

### Step 1: メールヘッダー解析

以下のフィールドを抽出する。

| フィールド | 抽出内容 | 備考 |
|-----------|---------|------|
| `From` | 送信者メールアドレス | 顧客ルックアップキー |
| `Subject` | 件名 | 分類・重複検出に利用 |
| `In-Reply-To` / `References` | スレッドID | 既存チケットへの追記判定 |
| `Date` | 受信日時（ISO 8601形式に正規化） | SLA開始タイムスタンプ |
| `Message-ID` | メール固有ID | チケットオブジェクトに記録 |

**スレッド検出ロジック**:
1. `In-Reply-To` ヘッダーが存在 → 同一値を持つ既存チケットを検索
2. 一致するチケットが存在 → 新規チケット作成せず、既存チケットに返信追記
3. 一致なし → 新規チケットとして処理を継続

### Step 2: 顧客情報ルックアップ

`From` メールアドレスを用いて顧客データベースを照合する。

**取得する顧客属性**:

| 属性 | 値域 | 後続処理への影響 |
|-----|------|---------------|
| `segment` | VIP / General / Complaint-in-progress | R1エスカレーション判定、Q3トーン制御 |
| `plan` | Free / Pro / Enterprise | T3 SLA算出 |
| `company` | 法人名（Enterprise の場合） | R3 パーソナライズ |
| `open_ticket_count` | 整数 | 重複チケット検出に利用 |
| `contact_history` | 直近対応履歴（最大5件） | R3 パーソナライズ、反復係数計算 |

**顧客不明時の処理**:
- データベースに該当なし → `segment: General`、`plan: Free` として処理を継続
- 不明フラグをチケットに付与し、T2 分類・T3 優先度算出時に参照

### Step 3: PII 初期検出・フラグ付与（v2 必須）

**重要**: この段階ではマスキングを行わない。PII の種別とチケット内の位置をフラグとして記録するのみ。マスキングは Q1（PII Scan）で実施する。

検出対象のPIIパターン（`common/content-validation.md` のパターンを参照）:

| PII種別 | 検出対象例 | フラグキー |
|--------|---------|----------|
| メールアドレス | `user@example.com` | `pii_email` |
| 電話番号 | `090-1234-5678`, `+81-90-1234-5678` | `pii_phone` |
| クレジットカード番号 | `4111-1111-1111-1111` | `pii_card` |
| マイナンバー | 12桁数字 | `pii_mynumber` |
| 住所 | 都道府県+市区町村+番地パターン | `pii_address` |
| IPアドレス | `192.168.x.x`, `10.x.x.x` 等 | `pii_ip` |

**フラグ記録形式**:
```
pii_flags: {
  pii_email: [{ location: "body", position: "line 3" }],
  pii_card: [{ location: "body", position: "line 7" }]
}
```

**社内スタッフPII**: 転送メールや CC フィールドに社内メールアドレスが含まれる場合も同様にフラグを付与する。

### Step 4: 感情シグナル検出（v2 必須）

メール本文のキーワードパターンから感情レベルを判定し、チケットに記録する。R1（Mode Selection）で参照される。

**感情分類ルール**:

| 感情レベル | 判定条件 | R1への影響 |
|----------|---------|---------|
| `angry`（怒り） | 「ふざけるな」「解約する」「訴える」「最悪」「詐欺」「謝罪しろ」等の強い否定表現 | `segment = Complaint-in-progress` との組み合わせで即エスカレーション |
| `frustrated`（不満） | 「まだ解決しない」「何度も連絡している」「早くしてほしい」等の継続不満表現 | R3 でクレーム対応構造を適用 |
| `confused`（困惑） | 「よくわからない」「どうすれば」「教えてほしい」等の質問・困惑表現 | R3 でステップバイステップ回答構造を適用 |
| `normal`（通常） | 上記パターンなし | 通常処理 |

**判定優先順位**: `angry` > `frustrated` > `confused` > `normal`（複数パターン合致時は最高レベルを採用）

### Step 5: 重複チケット検出

同一顧客からの24時間以内の類似問い合わせを検出する。

**検出ロジック**:
1. 同一 `From` メールアドレスで受信日時が24時間以内のオープンチケットを検索
2. 件名の類似度（Jaccard係数など）が 0.6 以上、またはカテゴリが同一 → 重複と判定
3. 重複検出時: 新規チケット作成せず、既存チケットの `repeat_count` をインクリメント（+1）
4. 既存チケットに本メールの本文を追記し、受信日時を更新

**反復係数（T3 優先度計算に利用）**:
- `repeat_count >= 2`: 優先度スコアに × 1.5 の乗数を適用（T3 で処理）

### Step 6: チケットオブジェクト生成

以下のテンプレートに従い、チケットオブジェクトを生成する。

```yaml
ticket:
  id: "TKT-{YYYYMMDD}-{連番}"
  created_at: "{ISO 8601タイムスタンプ}"
  status: "Open"

  email:
    message_id: "{Message-ID}"
    from: "{送信者メールアドレス}"
    subject: "{件名}"
    received_at: "{受信日時}"
    thread_id: "{In-Reply-Toの値 or null}"

  customer:
    email: "{メールアドレス}"
    segment: "{VIP | General | Complaint-in-progress | unknown}"
    plan: "{Free | Pro | Enterprise | unknown}"
    company: "{法人名 or null}"
    open_ticket_count: "{整数}"

  pii_flags:
    pii_email: []
    pii_phone: []
    pii_card: []
    pii_mynumber: []
    pii_address: []
    pii_ip: []

  emotion_signal: "{angry | frustrated | confused | normal}"
  repeat_count: "{整数}"
  is_duplicate: "{true | false}"
  parent_ticket_id: "{親チケットID or null}"

  audit_log:
    - timestamp: "{ISO 8601}"
      stage: "T1"
      action: "ticket_created"
      details: "PII flags: {pii_flags summary}, emotion: {emotion_signal}"
```

---

## Question Generation

T1 完了後、ユーザー確認が不要なため Question Generation は発生しない。  
ただし以下のケースでは処理を一時停止し、オペレーターに確認を求める。

- 顧客情報が完全に不明で、セキュリティインシデントの可能性がある場合
- メールが自動送信システムからのものと疑われ、ループ検出リスクがある場合

---

## Output Artifacts

T1 が生成するアーティファクト:

1. **チケットオブジェクト**（YAML形式、上記テンプレート）— T2/T3/R1 で参照
2. **PIIフラグセット** — Q1（PII Scan）で参照、マスキング実施時の位置情報として利用
3. **感情シグナル** — R1（Mode Selection）、Q3（Tone Calibration）で参照
4. **監査ログエントリ** — `stage: T1, action: ticket_created`

---

## Completion Message

T1 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED
チケットオブジェクトを生成しました。

- Ticket ID: TKT-20260405-001
- 顧客: {顧客メールアドレス} / セグメント: {VIP|General|Complaint-in-progress} / プラン: {Free|Pro|Enterprise}
- 感情シグナル: {angry|frustrated|confused|normal}
- PII検出: {検出した PII 種別リスト、または「なし」}
- 重複チケット: {「あり — TKT-xxx に追記」または「なし」}

### WHAT'S NEXT
**A) 修正依頼** — チケット情報を修正する
**B) T2（Classification）へ続行** — 問い合わせカテゴリの自動判定へ進む
```

---

## Error Handling

### エラー 1: スレッド検出漏れ（Pitfall: エスカレーション判定ミスの原因）

**症状**: `In-Reply-To` ヘッダーが存在するにもかかわらず、新規チケットとして登録してしまう。  
**原因**: `In-Reply-To` の照合対象が限定されている（例: 同日受信チケットのみ検索）。  
**対処**: 照合範囲を最低30日に拡張。`References` ヘッダーも照合対象に含める。

**GOOD例**:
```
In-Reply-To: <abc123@mail.example.com> が存在
→ 既存チケット TKT-20260401-010 の References に一致
→ TKT-20260401-010 に追記、repeat_count: 2 に更新
→ T3 で反復係数 ×1.5 適用
```

**BAD例**:
```
In-Reply-To: <abc123@mail.example.com> が存在するが検索範囲が7日のため見つからず
→ TKT-20260405-003 として新規登録
→ 顧客が「2回目の問い合わせ」と述べているにもかかわらず repeat_count: 0
→ T3 で反復係数が適用されず、優先度が過少評価される
```

### エラー 2: PII フラグ付与漏れ（Pitfall: PII漏洩の前段階リスク）

**症状**: メール本文にクレジットカード番号が含まれているが `pii_card` フラグが付与されない。  
**原因**: ハイフンなし形式（`4111111111111111`）のカード番号がパターン未登録。  
**対処**: ハイフンあり・なし・スペース区切りの3形式すべてをカバーする正規表現を `common/content-validation.md` に定義し、T1 はそれを参照する。

**GOOD例**:
```
本文: "カード番号: 4111-1111-1111-1111"
→ pii_card フラグ付与 (location: body, position: line 5)
→ Q1 で「****-****-****-1111」にマスキング
```

**BAD例**:
```
本文: "カード番号: 4111111111111111" (ハイフンなし)
→ pii_card フラグ未付与
→ Q1 でマスキング漏れ
→ 回答ドラフトにカード番号がそのまま含まれる
```

### エラー 3: 感情シグナル誤判定（Pitfall: トーンミスマッチ）

**症状**: 技術的な障害報告（「エラーが出る、最悪だ」）を `angry` と判定し、不要なエスカレーションが発生。  
**原因**: ネガティブ語の単純マッチ。コンテキストを考慮しない。  
**対処**: 感情キーワードの前後3文字のコンテキストを確認し、製品・サービスに向けた不満表現か、状況を説明する表現かを区別する。判定に自信がない場合は `confused` にフォールバックする。

**GOOD例**:
```
本文: "エラーコード 500 が出て最悪な状況です。至急確認をお願いします。"
→ 「最悪」はエラー状況の説明と判断
→ emotion_signal: frustrated（継続障害への不満）
→ R3 でクレーム対応構造適用（穏当な対処）
```

**BAD例**:
```
本文: "エラーコード 500 が出て最悪な状況です。至急確認をお願いします。"
→ 「最悪」の単純マッチで emotion_signal: angry と判定
→ segment: General であっても R1 で即エスカレーション候補になる
→ 本来の技術的問題への対処が遅延する
```

### エラー 4: 顧客情報不明時の誤ったデフォルト適用（Pitfall: エスカレーション判定ミス）

**症状**: Enterprise 顧客が別のメールアドレスから問い合わせ → 未知顧客として `plan: Free` が適用され、SLA が 24h に設定される。  
**対処**: 顧客不明フラグ付きチケットは T3 でオペレーターへの確認ステップを追加する。件名や本文に法人名・契約IDが含まれている場合は顧客ルックアップを再試行する。
