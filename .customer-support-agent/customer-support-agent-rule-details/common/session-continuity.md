# Session Continuity — チケット状態管理テンプレート

## このファイルの目的

エージェントがフェーズをまたいで一貫したチケット処理を行えるよう、チケット状態オブジェクトの構造・更新タイミング・引き継ぎルールを定義する。
`core-workflow.md` の各フェーズ（T1〜F2）で生成・更新されるチケット状態を管理する。

---

## チケット状態オブジェクト（Ticket State Object）

### 完全テンプレート

```json
{
  "ticket_id": "TKT-{YYYYMMDD}-{連番3桁}",
  "created_at": "{ISO 8601 timestamp}",
  "updated_at": "{ISO 8601 timestamp}",

  "customer": {
    "email": "***@***.com",
    "name": "{顧客名}",
    "segment": "VIP | General | Complaint-in-progress",
    "plan": "Free | Pro | Enterprise",
    "inquiry_history_count": 0
  },

  "triage": {
    "category": "Account | Billing | Feature | Bug | Security",
    "subcategory": "{サブカテゴリ}",
    "classification_confidence": 0.0,
    "priority": "P1 | P2 | P3 | P4",
    "priority_score": 0.0,
    "emotion_signal": "normal | confused | frustrated | angry",
    "duplicate_of": null,
    "repeat_count": 0
  },

  "sla": {
    "frt_target": "{ISO 8601 timestamp}",
    "frt_actual": null,
    "sla_status": "within | warning | exceeded",
    "sla_remaining_minutes": 0
  },

  "pii": {
    "detected_types": [],
    "initial_scan_at": "{ISO 8601 timestamp}",
    "masked": false
  },

  "kb": {
    "search_performed": false,
    "confidence_score": null,
    "articles_referenced": [],
    "freshness_warning": false
  },

  "response": {
    "mode": "immediate | escalation | analysis | pending",
    "draft_generated": false,
    "draft_approved": false,
    "round_trips": 0
  },

  "escalation": {
    "triggered": false,
    "trigger_reason": null,
    "target_team": null,
    "handoff_summary_id": null,
    "interim_response_sent": false
  },

  "quality": {
    "q1_pii_clear": false,
    "q2_content_verified": false,
    "q3_tone_calibrated": false,
    "q3_repair_count": 0,
    "q4_dispatched": false
  },

  "follow_up": {
    "status": "Open | Pending | Escalated | Resolved | Closed",
    "next_action_at": null,
    "close_reason": null
  },

  "audit_log_entries": []
}
```

---

## 状態更新タイミング

| ステージ | 更新フィールド |
|---------|-------------|
| T1: チケット受付 | `ticket_id`, `created_at`, `customer.*`, `triage.emotion_signal`, `pii.detected_types`, `pii.initial_scan_at`, `triage.duplicate_of`, `triage.repeat_count` |
| T2: 分類 | `triage.category`, `triage.subcategory`, `triage.classification_confidence` |
| T3: 優先度判定 | `triage.priority`, `triage.priority_score`, `sla.*` |
| R1: モード選択 | `response.mode` |
| R2: KB検索 | `kb.*` |
| R3: ドラフト生成 | `response.draft_generated`, `response.draft_approved` |
| R4: エスカレーション | `escalation.*`, `follow_up.status` |
| Q1: PIIスキャン | `pii.masked`, `quality.q1_pii_clear` |
| Q2: コンテンツレビュー | `quality.q2_content_verified` |
| Q3: トーンキャリブレーション | `quality.q3_tone_calibrated`, `quality.q3_repair_count` |
| Q4: ディスパッチ | `quality.q4_dispatched`, `sla.frt_actual`, `sla.sla_status` |
| F1: ケーストラッキング | `follow_up.*` |

`updated_at` はすべてのステージ更新時に現在時刻で更新する。

---

## 感情シグナル検出ルール（T1）

| シグナル | 検出キーワード例 | 対応アクション |
|---------|--------------|-------------|
| `normal` | （特定シグナルなし） | 標準処理 |
| `confused` | 「わからない」「教えてください」「どうすれば」「使い方」 | 丁寧な説明モード |
| `frustrated` | 「困っている」「何度も」「まだ解決しない」「ずっと待っている」 | 共感文追加、優先度上げ |
| `angry` | 「ありえない」「最悪」「解約する」「訴える」「クレームを入れる」 | エスカレーション候補フラグ |

`angry` × `Complaint-in-progress` セグメント → R1で即エスカレーション（`core-workflow.md` R1 感情ベース条件）

---

## チケットステータス遷移

```
Open（受付）
  │
  ├─ 即時回答送信 → Resolved（解決）→ Closed（7日後自動クローズ or 顧客確認）
  │
  ├─ 確認回答送信 → Pending（顧客返信待ち）→ Open（返信後）or Closed（7日無返信）
  │
  └─ エスカレーション → Escalated → Resolved（オペレーター解決報告後）→ Closed
```

### クローズ条件

- 顧客が解決を確認した
- 7日間返信なし（自動クローズ）
- オペレーターから解決報告を受けた

---

## エスカレーション時の引き継ぎ情報

エスカレーション（R4）時に、以下を引き継ぎサマリーとして生成する。

```
【チケット引き継ぎサマリー】
チケットID: {ticket_id}
顧客セグメント: {segment} / プラン: {plan}
問い合わせカテゴリ: {category} - {subcategory}
感情シグナル: {emotion_signal}
往復回数: {round_trips}
SLAステータス: {sla_status}

【対応履歴】
- 実施済みKB検索: {articles_referenced}
- 試行済み解決策: {actions_taken}

【エスカレーション理由】
{trigger_reason}

【推奨アクション】
{recommended_action}
```

---

## GOOD / BAD 例

### GOOD: セッション状態の引き継ぎ（正しい）

```
フェーズをまたぐ際、チケット状態オブジェクトを参照する。
例: Q3→R4 リペアパス時、quality.q3_repair_count を確認し
   1を超えていれば強制エスカレーションへ移行する（最大1回制限）。
```

### BAD: セッション状態の無視（誤り）

```
Q3→R4 リペアパス後、repair_count を更新しないまま再度 Q3→R4 を実行する。
```
理由: リペアループ上限（最大1回）を超える無限ループが発生する。

---

### GOOD: 重複チケット検出（正しい）

```
同一顧客から24時間以内に同一内容の問い合わせを受信 →
既存チケットに追記し、repeat_count を+1する。新規チケットは作成しない。
```

### BAD: 重複チケット作成（誤り）

```
同一顧客から同一内容の問い合わせを受信するたびに新規チケットを作成する。
```
理由: 対応履歴が分断され、repeat_count の閾値（2回以上で優先度×1.5）が正しく機能しない。

---

## 参照先

- `core-workflow.md` T1〜F1 — 各ステージでの状態更新詳細
- `common/error-handling.md` — 状態異常時のリカバリ
- `common/tone-guidelines.md` — 感情シグナル別トーン適用
