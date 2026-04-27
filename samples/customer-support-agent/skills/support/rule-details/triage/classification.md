# T2: Classification — Phase Rule

## Purpose

T1 で生成されたチケットオブジェクトをもとに、問い合わせカテゴリ・サブカテゴリを自動判定する。キーワード解析とコンテキスト分析を組み合わせて分類精度を高め、複数カテゴリ合致時の優先ルールを適用する。低信頼度の場合は手動分類フォールバックを発動する。

**Classification**: ALWAYS EXECUTE（全問い合わせで必須）
**Approval Gate**: なし（次ステージ T3 へ自動移行）
**参照元**: `core-workflow.md` T2 セクション

---

## Prerequisites

- T1 が完了しており、チケットオブジェクトが生成されていること
- `common/terminology.md` を読み込み済み（カテゴリ用語の参照元）
- カテゴリ別キーワード辞書が利用可能であること

---

## Execution Steps

### Step 1: カテゴリ体系の確認

以下の5カテゴリおよびサブカテゴリに分類する。

#### Account（アカウント）
| サブカテゴリ | 代表的な問い合わせ内容 |
|-----------|-----------------|
| `login_failure` | ログインできない、パスワードを忘れた |
| `account_locked` | アカウントがロックされた |
| `profile_update` | メールアドレス変更、プロフィール更新 |
| `account_deletion` | アカウント削除・退会手続き |
| `mfa_issue` | 二段階認証が機能しない |

#### Billing（課金）
| サブカテゴリ | 代表的な問い合わせ内容 |
|-----------|-----------------|
| `invoice_inquiry` | 請求書の確認・ダウンロード |
| `payment_failure` | 支払いが失敗した、カードが通らない |
| `plan_change` | プランのアップグレード・ダウングレード |
| `refund_request` | 返金・キャンセル手続き |
| `billing_error` | 請求金額の誤り |

#### Feature（機能・使い方）
| サブカテゴリ | 代表的な問い合わせ内容 |
|-----------|-----------------|
| `how_to_use` | 機能の使い方・操作方法 |
| `feature_request` | 新機能のリクエスト |
| `integration` | 外部サービスとの連携設定 |
| `api_usage` | API の仕様・使い方 |
| `data_export` | データエクスポート・インポート |

#### Bug（不具合）
| サブカテゴリ | 代表的な問い合わせ内容 |
|-----------|-----------------|
| `ui_bug` | 画面表示の崩れ、ボタンが機能しない |
| `performance` | 動作が遅い、タイムアウトが発生する |
| `data_inconsistency` | データが正しく保存・表示されない |
| `api_error` | APIがエラーを返す |
| `regression` | アップデート後に動作しなくなった |

#### Security（セキュリティ）
| サブカテゴリ | 代表的な問い合わせ内容 |
|-----------|-----------------|
| `unauthorized_access` | 不正アクセスの疑い、見知らぬログイン |
| `data_breach` | データ漏洩の可能性 |
| `phishing` | フィッシングメールの報告 |
| `vulnerability` | セキュリティ脆弱性の報告 |
| `compliance` | GDPRや個人情報保護法への対応確認 |

### Step 2: キーワード＋コンテキスト分類

**フェーズ 1: キーワードスキャン**

件名と本文を対象に、各カテゴリのキーワード辞書と照合する。

キーワード辞書の優先照合順序:
1. 件名（Subject）— 本文より高い重みを付与（×1.5）
2. 本文冒頭100文字 — 高重み（×1.2）
3. 本文全体 — 標準重み（×1.0）

**フェーズ 2: コンテキスト分析**

キーワードのみで判定が困難な場合、以下のコンテキスト情報を加味する。

| コンテキスト情報 | 利用目的 |
|--------------|---------|
| T1 で取得した顧客プラン | Billing サブカテゴリの絞り込み |
| T1 で取得した対応履歴 | 繰り返し問い合わせのカテゴリ継続性確認 |
| 添付ファイルの有無 | スクリーンショット添付 → Bug の可能性が高い |
| メール送信時刻 | 深夜の緊急問い合わせ → Security または Bug（P1）可能性 |

**フェーズ 3: 信頼度スコア算出**

```
分類信頼度スコア = (マッチキーワード数 × 重み合計) / 全キーワード検査数
```

- スコア >= 0.7: 高信頼度、自動分類を確定
- スコア 0.5-0.7: 中信頼度、分類確定するが T3 完了後のトリアージサマリーに注記
- スコア < 0.5: 低信頼度、手動分類フォールバックを発動

### Step 3: 複数カテゴリ合致時の優先ルール

複数カテゴリのキーワードが合致した場合、以下の優先順位でカテゴリを確定する。

```
Security > Bug > Billing > Feature > Account
```

**優先ルール適用例**:
- 「ログインできない、不正アクセスの疑いがある」→ Account と Security が合致 → **Security** を選択
- 「請求画面でエラーが出る」→ Billing と Bug が合致 → **Bug** を選択
- 「機能が遅い、課金されているのに使えない」→ Bug と Billing が合致 → **Bug** を選択

**サブカテゴリの扱い**: プライマリカテゴリを確定後、サブカテゴリを最大2件まで付与できる（複合問い合わせの場合）。

### Step 4: 低信頼度フォールバック

分類信頼度スコアが 0.5 未満の場合:

1. 最も高いスコアのカテゴリを候補として記録する（確定はしない）
2. チケットに `classification_status: pending_manual` フラグを付与する
3. T3 完了後、CP-1 トリアージサマリーにて「分類要確認」として提示する
4. オペレーターが手動でカテゴリを確定する

**自動エスカレーション例外**: 低信頼度でも、件名または本文に以下が含まれる場合は Security カテゴリを強制付与する。
- 「不正ログイン」「unauthorized」「data breach」「漏洩」「hack」「脆弱性」

### Step 5: 分類結果のチケットへの記録

```yaml
classification:
  primary_category: "{Account | Billing | Feature | Bug | Security}"
  subcategories:
    - "{サブカテゴリ1}"
    - "{サブカテゴリ2（オプション）}"
  confidence_score: "{0.0 - 1.0}"
  classification_status: "{confirmed | pending_manual}"
  matched_keywords: ["{キーワードリスト}"]
  priority_rule_applied: "{true | false}"
  priority_rule_reason: "{適用理由（複数カテゴリ合致の場合）}"

audit_log:
  - timestamp: "{ISO 8601}"
    stage: "T2"
    action: "classified"
    details: "category: {category}, confidence: {score}, status: {confirmed|pending_manual}"
```

---

## Question Generation

T2 完了後の Question Generation は発生しない。ただし `classification_status: pending_manual` の場合は CP-1 で確認を求める（T3 のトリアージサマリーに組み込む）。

---

## Output Artifacts

T2 が生成するアーティファクト:

1. **分類結果**（チケットオブジェクトの `classification` フィールドに追記）— T3/R1 で参照
2. **監査ログエントリ** — `stage: T2, action: classified`

---

## Completion Message

```
### REVIEW REQUIRED
問い合わせカテゴリを判定しました。

- カテゴリ: {Primary Category} / {Subcategory}
- 分類信頼度: {スコア}
- 分類ステータス: {confirmed | pending_manual（手動確認が必要）}
- 優先ルール適用: {なし | 「{適用理由}」により Security を優先適用}

### WHAT'S NEXT
**A) 修正依頼** — 分類を修正する
**B) T3（Priority Assessment）へ続行** — 優先度スコアリングへ進む
```

---

## Error Handling

### エラー 1: 課金問題をアカウント問題に誤分類（Pitfall: トーンミスマッチ・エスカレーション判定ミス）

**症状**: 「支払いが失敗してログインできない」→ `Account/login_failure` に分類。  
**原因**: 「ログインできない」というキーワードが Account に高スコアでマッチ。支払い失敗による機能制限という複合コンテキストを見落とし。  
**対処**: Billing の `payment_failure` キーワードが存在する場合、Account のログイン関連サブカテゴリとの複合として検出し、Billing を優先カテゴリとして採用する。

**GOOD例**:
```
件名: "ログインができません"
本文: "先日の支払いが失敗したと通知が来てから、ログインできなくなりました。"
→ "支払いが失敗" → Billing/payment_failure マッチ (重み: 1.2)
→ "ログインできない" → Account/login_failure マッチ (重み: 1.0)
→ 優先ルール: Billing > Account は成立しないが、Billingが原因のAccount問題と判定
→ primary_category: Billing, subcategory: [payment_failure, login_failure]
```

**BAD例**:
```
件名: "ログインができません"
本文: "先日の支払いが失敗したと通知が来てから、ログインできなくなりました。"
→ "ログインできない" のみにマッチ → Account/login_failure に単純分類
→ Billing チームへのルーティング漏れ → 顧客の課金問題が未解決
```

### エラー 2: セキュリティ関連キーワードの見落とし（Pitfall: エスカレーション判定ミス）

**症状**: 「見知らぬ国からのアクセス通知が届いた」→ `Feature/how_to_use` に分類。  
**原因**: 「通知」「アクセス」がセキュリティキーワード辞書に不足。  
**対処**: Security カテゴリのキーワード辞書に「見知らぬ」「知らない場所から」「心当たりのない」「不審な」などのコンテキスト表現を追加する。低信頼度でも Security キーワードが1件以上ある場合は Security を強制付与する。

**GOOD例**:
```
件名: "ログイン通知について"
本文: "見知らぬ場所からのログイン通知が届きましたが、心当たりがありません。"
→ "見知らぬ" + "心当たりがありません" → Security/unauthorized_access マッチ
→ 低信頼度でも Security 強制付与ルール適用
→ primary_category: Security, subcategory: [unauthorized_access]
→ T3 で P1/1h SLA が付与される
```

**BAD例**:
```
件名: "ログイン通知について"
本文: "見知らぬ場所からのログイン通知が届きましたが、心当たりがありません。"
→ "ログイン通知" → Account/login_failure としてマッチ
→ Security 強制付与ルールが未実装
→ primary_category: Account → T3 で P3/24h SLA
→ 不正アクセス対応が24時間遅延
```

### エラー 3: 分類信頼度が低いまま確定してしまう（Pitfall: ハルシネーション回答）

**症状**: 信頼度スコア 0.3 の状態で `Feature/how_to_use` として確定 → R2 で見当違いのKB記事を参照。  
**原因**: 低信頼度フォールバックが実装されていない。  
**対処**: スコア < 0.5 で必ず `pending_manual` フラグを付与し、CP-1 でオペレーター確認を経てからカテゴリ確定する。R1 以降は確定済みカテゴリのみを参照する。
