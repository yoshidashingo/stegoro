# A2: Action Identification — Phase Rule

## Purpose

S4の検出結果リストとA1のヘルススコアから、具体的な対応アクションを生成する。7種の検出種別に対応するアクション生成ルールを適用し、矛盾フラグ付き案件は「人間確認依頼」アクションに変換する。1商談1アクション原則（Pitfall 3対策）を徹底し、コンテキスト付きのアクションオブジェクトを確定する。

**Classification**: ALWAYS EXECUTE（日中モード・対応要案件あり時に必須）
**Approval Gate**: なし（次ステージ A3 へ自動移行）
**参照元**: `workflow-architecture.md` A2 セクション

---

## Prerequisites

- S4の検出結果リスト（`state/detection-results.yaml`）が確定していること
- A1のヘルススコアリスト（`state/health-scores.yaml`）が参照可能であること
- S3の統合ビュー（`state/integrated-view.yaml`）が参照可能であること（コンテキスト生成用）
- アクション生成テンプレート（`config/action-templates.yaml`）が設定済みであること

---

## Execution Classification

**ALWAYS EXECUTE**

- Execute IF: S4の検出結果リストに1件以上の案件が存在する場合
- Skip IF: S4で検出件数0件（このステージには到達しない）

---

## Execution Steps

### Step 1: 検出種別ごとのアクション生成ルール適用

各検出エントリの `type` に対応するアクションを生成する。

**アクション生成マッピングテーブル**:

| 検出種別 | 生成アクション（主） | 生成アクション（副） | 権限分類 |
|---------|-----------------|-----------------|---------|
| `stale_deal` | ステータス確認リマインダー送信（担当者Slack DM） | Salesforce Activity記録（自動） | 主: 自動 / 副: 自動 |
| `unanswered_email` | フォローアップメール送信（顧客へ） | Salesforce Activity記録（自動） | 主: **承認制** / 副: 自動 |
| `meeting_prep` | 会議準備サマリー生成・担当者送付 | Salesforce Activity記録（自動） | 主: 自動 / 副: 自動 |
| `amount_changed` | 金額変更確認通知（担当者へ） | — | **承認制** |
| `deadline_approaching` | クロージング期限アラート（担当者） | ディール加速提案（承認制、P1のみ） | 主: 自動 / 副: **承認制** |
| `untracked_lead` | リード登録提案 | — | **承認制** |
| `status_not_recorded` | Salesforce Activity自動記録 | ステージ更新提案 | 主: 自動 / 副: **承認制** |
| `conflict_review` | 担当者確認依頼通知 | — | **承認制** |

### Step 2: アクションオブジェクトの生成

各検出エントリに対応するアクションオブジェクトを生成する。

**アクションオブジェクトのフォーマット**:

```yaml
action:
  action_id: "ACT-20260405-001"
  detection_type: "stale_deal"
  action_type: "send_reminder"        # send_reminder / send_email / record_activity /
                                       # update_stage / generate_meeting_prep /
                                       # confirm_amount / register_lead / human_review
  target_opportunity_id: "006xxxxxxxxxxxx"
  target_opportunity_name: "株式会社B — ライセンス更新"
  authority: "auto"                    # auto / approval_required
  priority: "high"                     # critical / high / medium / low
  health_score: 42
  health_color: "yellow"

  context: |
    商談: 株式会社B — ライセンス更新
    金額: ¥4,200,000 / ステージ: Proposal / クロージング予定: 2026-04-30（25日後）
    検出理由: Proposalステージに16日間停滞（閾値: 14日）
    最終アクティビティ: 2026-03-20のメール送信
    Gmail/Slack/Calendar: 直近7日のアクティビティなし

  payload:
    recipient_slack_id: "U123456789"
    message_template: "stale_deal_reminder"
    additional_context:
      days_in_stage: 16
      last_activity_date: "2026-03-20"
```

**アクションIDの発行規則**: `ACT-{YYYYMMDD}-{3桁連番}`（サイクル内でユニーク）

### Step 3: 各検出種別のアクション詳細生成

#### 停滞商談（stale_deal）のアクション

主アクション — ステータス確認リマインダー（自動実行）:
```yaml
action_type: send_reminder
authority: auto
payload:
  channel: slack_dm
  recipient: {担当者Slack ID}
  message: |
    【パイプライン確認依頼】{商談名}

    {商談名}（¥{金額}、{ステージ}ステージ）が{days_in_stage}日間進捗していません。
    最終アクティビティ: {last_activity_date}

    現在の状況を教えていただけますか？
    Salesforceの更新が必要な場合はお知らせします。
```

副アクション — Activity自動記録（自動実行）:
```yaml
action_type: record_activity
authority: auto
payload:
  sf_object: Task
  subject: "[エージェント記録] 停滞検出 — {ステージ}ステージ {days_in_stage}日経過"
  activity_date: {today}
  status: Completed
```

#### フォローアップメール送信（unanswered_email）のアクション

```yaml
action_type: send_email
authority: approval_required
payload:
  recipient_email: {顧客メールアドレス}
  thread_id: {元スレッドID}
  subject_prefix: "Re: "
  draft_content: |
    {担当者名}様

    先日ご送付した「{件名}」について、その後いかがでしょうか。
    ご確認いただけましたら、ご意見・ご質問等お気軽にお知らせください。

    引き続きよろしくお願いいたします。
  approval_note: "顧客への送信前に内容をご確認ください。文面は修正可能です。"
```

#### 会議準備サマリー生成（meeting_prep）のアクション

```yaml
action_type: generate_meeting_prep
authority: auto
payload:
  event_id: {カレンダーイベントID}
  meeting_start: {ミーティング開始時刻}
  summary_template: |
    ## {ミーティング件名} — 会議準備サマリー

    ### 商談概要
    - 商談名: {opportunity_name}
    - 金額: ¥{amount} / ステージ: {stage} / クロージング予定: {close_date}
    - ヘルススコア: {health_color}（{health_score}点）

    ### 直近のやり取り（過去14日）
    {gmail_activities_summary}
    {slack_mentions_summary}

    ### 未解決の懸念事項
    {open_concerns_from_context}

    ### 推奨アジェンダ
    {suggested_agenda_items}
  delivery_channel: slack_dm
  recipient: {担当者Slack ID}
```

#### 未追跡リード登録提案（untracked_lead）のアクション

```yaml
action_type: register_lead
authority: approval_required
payload:
  email: {見込み先メールアドレス}
  domain: {見込み先ドメイン}
  contact_date: {最初のコンタクト日}
  source_evidence: {Gmailメッセージ要約}
  proposed_lead_record:
    Email: {email}
    Company: {ドメインから推定した社名}
    LeadSource: "Gmail"
    Status: "Open - Not Contacted"
  approval_note: "新規リード登録の内容をご確認ください。情報を修正して承認してください。"
```

### Step 4: コンテキストの付与

全アクションに判断根拠となるコンテキスト情報を付与する。コンテキストは承認担当者が判断するために必要な最小限の情報を含む。

**コンテキスト必須フィールド**:
- 商談名・金額・ステージ・クロージング日
- 検出理由（何日停滞、何日未返信など具体的な数値）
- 直前のアクティビティ情報（最終メール件名・日付、最終Slack言及内容）
- ヘルススコアとカラー

**コンテキスト生成例**:
```
商談: 株式会社D — SaaS基盤移行プロジェクト
金額: ¥12,000,000 / ステージ: Negotiation / クロージング予定: 2026-04-15（10日後）
検出理由: Negotiationステージに8日間停滞（閾値: 7日）
最終アクティビティ: 2026-03-28のSlackメッセージ「契約条件について週明けに連絡する」
ヘルススコア: 赤（35点）— 期限接近かつ長期停滞
```

### Step 5: 1商談1アクション原則の最終確認（Pitfall 3対策）

全アクション生成後、同一 `target_opportunity_id` を持つアクションが複数存在する場合は統合する。

**統合ルール**:
1. 最高権限分類のアクションを「主アクション」として保持（承認制 > 自動）
2. 同一商談の他アクションは `additional_actions` リストに移動
3. 自動実行アクションは `immediate_auto_actions` として別途記録（主アクションの承認とは独立して即時実行可能）

```yaml
# 統合例
action:
  action_id: "ACT-20260405-007"
  action_type: "send_email"         # 主アクション（承認制）
  authority: approval_required
  target_opportunity_id: "006xxx"

  immediate_auto_actions:           # 承認不要で即実行する副アクション
    - action_type: record_activity
      authority: auto

  additional_context_from_detections:
    - "Proposalステージ14日停滞"
    - "期限まで残5日"
```

### Step 6: アクションリストの確定

全アクションを生成・統合し、アクションリストファイルに書き出す。

```yaml
action_list:
  generated_at: "2026-04-05T10:05:00+09:00"
  cycle_id: "cycle-uuid-xxxx"
  total_actions: 8
  auto_executable: 5
  approval_required: 3

  actions:
    - action_id: "ACT-20260405-001"
      # ... 各アクション
```

---

## Output Artifacts

A2 が生成するアーティファクト:

1. **アクションリスト**（`state/action-list.yaml`） — A3・A4・D1・D2で参照
2. **自動実行アクション数・承認制アクション数サマリー** — サイクル状態ファイルに追記
3. **コンテキスト付きアクションオブジェクト** — D2の承認リクエスト生成に使用

---

## Completion Message

A2 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — A2: Action Identification 完了

生成アクション（合計: X件）:
- 自動実行: [X]件
  - Activity記録: [X]件
  - リマインダー送信: [X]件
  - 会議準備サマリー生成: [X]件
- 承認制: [X]件
  - フォローアップメール送信: [X]件
  - リード登録提案: [X]件
  - 人間確認依頼: [X]件

1商談1アクション原則により統合: [X]件

### WHAT'S NEXT
**A) 修正依頼** — アクション生成ルールやテンプレートを修正する
**B) [A3 Priority Ranking]へ続行** — アクション優先度スコアリングへ進む
```

---

## Error Handling

### エラー 1: 1商談に複数の承認リクエストを生成（Pitfall 3対策）

**症状**: 商談Eに「フォローアップメール送信」「ステージ更新提案」「リード商談化」の3件の承認リクエストを個別に生成し、担当者が同一商談について3件の承認リクエストを受け取る。
**原因**: Step 5の1商談1アクション統合処理が実装されていない。
**対処**: 同一 `target_opportunity_id` の承認制アクションは必ず1件の主アクションに統合し、他は `additional_context` として付加する。

**GOOD例**:
```
商談E: unanswered_email + status_not_recorded + stale_deal の3検出
→ アクション生成: send_email（承認制）+ record_activity（自動）+ send_reminder（自動）
→ 権限分類で統合: send_email を主アクション（承認制）に設定
→ record_activity と send_reminder は immediate_auto_actions に移動
→ 担当者への承認リクエスト: 1件（コンテキストに停滞・未返信の情報を付加）
（Pitfall 3対策）
```

**BAD例**:
```
商談E: unanswered_email + status_not_recorded + stale_deal の3検出
→ それぞれ個別にアクション生成
→ 承認リクエスト3件を個別に担当者Slack DM送信
→ 担当者が同一商談に3件の承認リクエストを受け取り困惑
→ 全て「却下」し以降の通知を無視するようになる（Pitfall 3対策漏れ）
```

### エラー 2: コンテキスト情報なしのアクション生成

**症状**: 承認リクエストに「フォローアップメール送信を承認してください」とだけ記載され、担当者がどの商談の何のメールかを判断できない。
**原因**: Step 4のコンテキスト付与処理をスキップした場合。
**対処**: コンテキスト必須フィールド（商談名・金額・ステージ・クロージング日・検出理由・直前アクティビティ）が全て揃っていることをアクション生成前に確認する。欠損がある場合はデフォルト値「不明」を記入するが、警告を監査ログに記録する。

**GOOD例**:
```
承認リクエスト:
【承認リクエスト】株式会社F — ERP導入
推奨アクション: フォローアップメール送信
理由: 「提案書確認のお願い」への返信が3営業日（4/1送信）ありません
商談詳細: 金額¥8,000,000 / Proposalステージ / クロージング予定4/30（25日後）
ヘルススコア: 黄（55点）
→ 担当者が1件の情報で承認/却下を判断可能
```

**BAD例**:
```
承認リクエスト:
「フォローアップメール送信を承認してください」（コンテキストなし）
→ 担当者: 「どの商談の話？どのメール？なぜ送るの？」と困惑
→ 全ての承認リクエストを却下
→ 承認制アクションが機能しなくなり、フォローアップが停滞
```

### エラー 3: 会議準備サマリーの情報不足

**症状**: 会議準備サマリーにSalesforce情報のみが含まれ、直近のGmail/Slackでの交渉内容が反映されていない。担当者が前回の交渉状況を把握せずにミーティングに臨む。
**原因**: meeting_prep アクションのペイロード生成時に統合ビューの `gmail_activities` および `slack_mentions` を参照していない。
**対処**: 会議準備サマリー生成時は統合ビューから Gmail・Slack・Calendar の全アクティビティを参照し、直近14日の交渉内容を必ず含める。

**GOOD例**:
```
会議準備サマリー生成: 株式会社G — 翌日10:00ミーティング
→ Salesforce情報: Negotiationステージ、¥6M、クロージング4/20
→ Gmail（直近14日）: 4/3「価格交渉メール送信」、4/4「顧客から追加要件の返信」
→ Slack（直近14日）: 4/5「担当者: 週明けに最終回答する予定」
→ サマリーに全情報を統合 → 担当者がコンテキストを把握してミーティングへ
```

**BAD例**:
```
会議準備サマリー: Salesforceデータのみ参照
→ 「Negotiationステージ、¥6M」の情報のみ
→ 直近の価格交渉メールや追加要件の情報なし
→ 担当者が前回交渉内容を把握せずにミーティングに参加
→ 顧客から「先週送った追加要件の回答は？」と聞かれて対応できない
```

### エラー 4: 夜間承認リクエストの即時送信制御漏れ（ドメイン固有）

**症状**: A2で生成したフォローアップメール送信アクションが夜間モード中に承認リクエストとして即時送信され、深夜に顧客へのメールが実行される。
**原因**: アクション生成時に時間帯チェックを実施せず、D2・D3での夜間制約チェックに依存している。
**対処**: A2のアクション生成時点で夜間モード（S1の判定結果を参照）であれば、顧客向け送信アクション（send_email）に `deferred_until: "next_business_day_08:00"` フラグを付与する。

**GOOD例**:
```
現在時刻: 夜間モード（21:00）
→ send_email アクション生成
→ deferred_until: "2026-04-06T08:00:00+09:00" フラグを付与
→ D2での承認リクエスト送信は実施（担当者は夜間でも承認可能）
→ 実際の顧客メール送信は翌朝8:00まで保留（夜間制約）
```

**BAD例**:
```
現在時刻: 夜間モード（21:00）
→ send_email アクション生成（夜間フラグなし）
→ D3で承認後に即時実行
→ 顧客に深夜21:30にフォローアップメールが届く
→ 顧客が不快感を示し商談の信頼関係が毀損（夜間制約違反）
```
