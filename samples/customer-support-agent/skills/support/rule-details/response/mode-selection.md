# R1: Mode Selection — Phase Rule

## Purpose

T1〜T3 のトリアージ結果をもとに、処理モードを選択する。条件ベースおよび感情ベースのエスカレーション条件を評価し、即時回答・エスカレーション・分析の3モードのいずれかを確定する。全モードで最高優先度の判定を行い、曖昧さなくルーティングする。

**Classification**: ALWAYS EXECUTE（全問い合わせで必須）
**Approval Gate**: なし（判定後ただちに該当ステージへ移行）
**参照元**: `core-workflow.md` R1 セクション

---

## Prerequisites

- T3 が完了し、CP-1 でトリアージ結果が承認されていること
- T1 の感情シグナル（`emotion_signal`）と顧客セグメント（`segment`）が確定していること
- T2 のプライマリカテゴリが確定していること（`classification_status: confirmed`）
- `common/tone-guidelines.md` を読み込み済み（感情ベースエスカレーション判定のトーンマトリクス参照）

---

## Execution Steps

### Step 1: 分析モード条件チェック

他の条件より前に分析モードを確認する（管理者リクエストが最優先で処理されるため）。

**分析モード発動条件**:
- 管理者からのレポートリクエストメッセージ（件名に「レポート依頼」「傾向分析」「週次サマリー」等）
- 定期レポートタイミングのシステムトリガー（週次・月次バッチ）

条件合致 → **分析モード確定** → R5（Trend Analysis）へ移行

分析モード以外の場合は Step 2 へ進む。

### Step 2: 条件ベースエスカレーション評価

以下の条件を上から順に評価する。1つでも合致した場合、即座にエスカレーションモードを確定し、残りの条件評価をスキップする。

| 優先度 | 条件 | 判定根拠 |
|------|-----|---------|
| 1 | カテゴリ = Security（任意サブカテゴリ） | T2 分類結果 |
| 2 | 本文にデータ損失リスクを示す表現（「データが消えた」「バックアップが取れない」「全件削除された」等） | T1 本文解析 |
| 3 | 本文に法的問題を示す表現（「弁護士に相談する」「法的措置」「訴訟」「個人情報保護委員会に通報」等） | T1 本文解析 |
| 4 | 顧客が明示的にオペレーターを要求（「担当者を出せ」「人間と話したい」「上司を呼べ」等） | T1 本文解析 |
| 5 | 同一チケット内の未解決往復が 3 回以上 | `ticket.repeat_count >= 3`（同一チケット） |
| 6 | チケット横断で同カテゴリが 3 件以上オープン | 顧客対応履歴の同カテゴリオープン数 |

条件合致 → **エスカレーションモード確定** → 合致した条件を `escalation_reason` に記録 → R4（Escalation Handoff）へ移行

### Step 3: 感情ベースエスカレーション評価（v2）

条件ベースで非該当だった場合、感情シグナルとセグメントの組み合わせを評価する。

**感情ベースエスカレーション発動条件**:

```
emotion_signal = "angry" AND segment = "Complaint-in-progress"
→ 即エスカレーション（tone-guidelines.md トーンマトリクスの「即エスカレーション」セルと同一条件）
```

この組み合わせが成立した場合 → **エスカレーションモード確定** → `escalation_reason: "emotion_based_angry_complaint"` を記録 → R4 へ移行

**その他の感情シグナルの扱い**:

| 感情シグナル | セグメント | 処理 |
|-----------|---------|-----|
| `angry` | General / VIP | 即時回答モードで継続（R3 でクレーム構造適用、Q3 でトーン厳格チェック） |
| `frustrated` | 任意 | 即時回答モードで継続（R3 でクレーム構造適用） |
| `confused` / `normal` | 任意 | 即時回答モードで継続（標準構造） |

### Step 4: デフォルト — 即時回答モード確定

Step 1〜3 のいずれの条件にも合致しない場合、即時回答モードを確定する。

**即時回答モード** → R2（Knowledge Retrieval）へ移行

### Step 5: モード選択結果の記録

```yaml
mode_selection:
  selected_mode: "{immediate | escalation | analysis}"
  escalation_reason: "{条件コードまたは null}"
  emotion_escalation: "{true | false}"
  analysis_trigger: "{manager_request | scheduled | null}"

audit_log:
  - timestamp: "{ISO 8601}"
    stage: "R1"
    action: "mode_selected"
    details: "mode: {mode}, reason: {escalation_reason or null}"
```

---

## 判定ツリー（全フロー）

```
R1 開始
│
├─ [分析モード条件に合致？]
│   YES → 分析モード → R5
│
├─ [条件ベースエスカレーション（6条件のいずれかに合致？）]
│   YES → エスカレーションモード → R4
│
├─ [感情ベースエスカレーション（angry × Complaint-in-progress）]
│   YES → エスカレーションモード → R4
│
└─ すべて非該当 → 即時回答モード → R2
```

---

## Question Generation

R1 は Approval Gate なし。ただし、エスカレーション判定の根拠が複数ある場合、CP-3（R4 完了後）で根拠リストを提示する。

---

## Output Artifacts

R1 が生成するアーティファクト:

1. **モード選択結果**（チケットオブジェクトの `mode_selection` フィールドに追記）— R2/R4/R5 のルーティングに利用
2. **エスカレーション理由**（エスカレーションモード時のみ）— R4 の引き継ぎサマリーに組み込む
3. **監査ログエントリ** — `stage: R1, action: mode_selected`

---

## Completion Message

```
### REVIEW REQUIRED
対応モードを選択しました。

- 選択モード: {即時回答 | エスカレーション | 分析}
- 判定根拠: {条件ベース: {条件名} | 感情ベース: angry × Complaint-in-progress | 分析トリガー: {種別} | デフォルト}

### WHAT'S NEXT
**A) 修正依頼** — モード選択を変更する
**B) {R2（Knowledge Retrieval）| R4（Escalation Handoff）| R5（Trend Analysis）}へ続行** — 次のステージへ進む
```

---

## Error Handling

### エラー 1: 「担当者を出せ」を見落とし即時回答（Pitfall: エスカレーション判定ミス）

**症状**: 本文に「人間の担当者と話したい」という明示的要求があるにもかかわらず、KB 検索・ドラフト生成へ進んでしまう。  
**原因**: 条件 4（明示的オペレーター要求）のキーワード辞書が不足。  
**対処**: 「担当者」「人間」「オペレーター」「上司」「責任者」「マネージャーを呼べ」等のパターンをキーワード辞書に網羅的に登録する。合致した場合は即座に `escalation_reason: "explicit_operator_request"` でエスカレーション確定する。

**GOOD例**:
```
本文: "AIの回答では解決しません。人間の担当者と直接話したいです。"
→ 条件4: "人間の担当者" にマッチ
→ エスカレーションモード確定
→ escalation_reason: "explicit_operator_request"
→ R4 で「担当者にご案内します」テンプレートを使用
```

**BAD例**:
```
本文: "AIの回答では解決しません。人間の担当者と直接話したいです。"
→ 「担当者」がキーワード辞書に未登録
→ 即時回答モード → R2 で KB 検索
→ KB から回答を生成して送信
→ 顧客: 「なぜまた AI からの回答が来るのか」とさらに激化
```

### エラー 2: 怒りシグナル × Complaint-in-progress の見落とし（Pitfall: トーンミスマッチ）

**症状**: `emotion_signal: angry` + `segment: Complaint-in-progress` の組み合わせを見落とし、即時回答モードで KB 回答を送信。  
**原因**: 感情ベースエスカレーション判定（Step 3）が未実装、または判定条件が `AND` ではなく `OR` になっている。  
**対処**: Step 3 の条件を `emotion_signal = "angry" AND segment = "Complaint-in-progress"` の厳密な論理積（AND）で実装する。どちらか一方のみでは発動しないことに注意する。

**GOOD例**:
```
顧客: segment = Complaint-in-progress / emotion_signal = angry
→ 感情ベースエスカレーション条件に合致（AND 条件成立）
→ エスカレーションモード確定
→ escalation_reason: "emotion_based_angry_complaint"
→ R4 で人間オペレーターへ引き継ぎ
```

**BAD例**:
```
顧客: segment = Complaint-in-progress / emotion_signal = angry
→ 感情ベース判定が OR 条件: angry OR Complaint-in-progress のいずれかで発動
→ 通常顧客が "angry" と判定されただけでエスカレーション多発
→ または条件が未実装で即時回答モードに進み、クレーム対応中顧客を AI 回答で処理
```

### エラー 3: 分析モードが先に評価されず、管理者リクエストが通常チケットとして処理される（Pitfall: エスカレーション判定ミス）

**症状**: 管理者からの週次レポート依頼メールが、「課金問い合わせ」として分類・処理されてしまう。  
**原因**: 分析モード条件を Step 1 で最初に評価していない。  
**対処**: 分析モードの評価を必ず Step 1 で最初に行い、合致した場合は他の条件評価をスキップして R5 へ移行する。
