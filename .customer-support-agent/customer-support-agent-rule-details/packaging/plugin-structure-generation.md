# P1: Plugin Structure Generation（プラグイン構造生成）

## Purpose

検証済みのカスタマーサポートエージェントポリシー一式をClaude Codeプラグイン形式にパッケージングする。plugin.json・marketplace.json・agent定義・skill定義・command定義を生成し、rule-detailsファイルをプラグイン構造にコピーする。このフェーズはポリシー初期構築時またはポリシー更新時のみ実行し、チケット処理のランタイムフローには含まれない。

**Classification**: ALWAYS EXECUTE

**参照元**: `core-workflow.md` P1（BUILD-TIME: PACKAGING）

---

## Prerequisites

- すべてのPhase Rules（16ファイル）とCommon Rules（12ファイル）、core-workflow.md（1ファイル）の計29ファイルが生成済みであること
- P2（Automated Validation）を事前実行する場合、P1生成後にP2が実行される順序であること
- P2→P1ループ回数カウンターが参照可能であること（最大2回）

---

## Execution Steps

### Step 1: REFINEMENTラウンド結果の読み込み

ポリシー生成プロセスの最終状態を確認する。

```
確認項目:
  □ 全29ファイルが存在すること（File Inventory: 29件）
  □ core-workflow.md内の全参照先ファイルが存在すること
  □ 直前のP2 FAILによるP1再実行の場合、失敗箇所のログを参照していること
```

### Step 2: plugin.json 生成

```json
{
  "name": "customer-support-agent",
  "version": "2.0.0",
  "description": "SaaS/Webサービス向けカスタマーサポートエージェント。メール問い合わせのトリアージ・回答生成・品質検査・フォローアップを自動化する。",
  "author": "steering-policy-generator",
  "license": "MIT",
  "entrypoint": "customer-support-agent-rules/core-workflow.md",
  "rule_details": "customer-support-agent-rule-details/",
  "agents": ["agents/customer-support-agent.json"],
  "skills": [
    "skills/triage.json",
    "skills/response.json",
    "skills/quality.json",
    "skills/follow-up.json"
  ],
  "commands": ["commands/support.json"],
  "requires": {
    "claude_code": ">=1.0.0"
  },
  "tags": ["customer-support", "email", "saas", "pii-protection", "sla-aware"]
}
```

### Step 3: agents/customer-support-agent.json 生成

```json
{
  "id": "customer-support-agent",
  "name": "カスタマーサポートエージェント",
  "description": "SaaSメールサポートの受付〜回答〜品質保証を一貫して処理するハイブリッドエージェント",
  "workflow": "core-workflow.md",
  "modes": ["immediate_response", "escalation", "analysis"],
  "segments": ["VIP", "General", "Complaint-in-progress"],
  "mandatory_phases": ["TRIAGE", "QUALITY"],
  "conditional_phases": ["FOLLOW-UP"],
  "build_time_phases": ["PACKAGING"],
  "pii_protection": "mandatory",
  "sla_awareness": true,
  "audit_log": "append-only"
}
```

### Step 4: skills/*.json 生成（4ファイル）

各フェーズをskillとして定義する。

**skills/triage.json**（例）:
```json
{
  "id": "triage",
  "name": "トリアージ",
  "phases": ["T1", "T2", "T3"],
  "classification": "ALWAYS",
  "rule_files": [
    "customer-support-agent-rule-details/triage/ticket-reception.md",
    "customer-support-agent-rule-details/triage/classification.md",
    "customer-support-agent-rule-details/triage/priority-assessment.md"
  ],
  "checkpoint": "CP-1"
}
```

skills/response.json、skills/quality.json、skills/follow-up.json も同形式で生成する（各フェーズのステージ・ルールファイル・チェックポイントを定義）。

### Step 5: commands/support.json 生成

```json
{
  "command": "support",
  "description": "カスタマーサポート処理を開始する",
  "usage": "/support [email_content]",
  "triggers": [
    "カスタマーサポート",
    "問い合わせ処理",
    "サポートメール"
  ],
  "invokes": "customer-support-agent"
}
```

### Step 6: rule-detailsファイルのコピー

生成済みの29ファイルを以下の構造でプラグインディレクトリにコピーする。

```text
.customer-support-agent/
├── customer-support-agent-rules/
│   └── core-workflow.md
└── customer-support-agent-rule-details/
    ├── common/         (12ファイル)
    ├── triage/         (3ファイル: T1, T2, T3)
    ├── response/       (5ファイル: R1-R5)
    ├── quality/        (4ファイル: Q1-Q4)
    ├── follow-up/      (2ファイル: F1, F2)
    └── packaging/      (2ファイル: P1, P2)
```

---

## Question Generation

CP-8として以下の確認を提示する。

```
### CP-8: プラグイン構造確認
生成されたプラグイン構造:
- plugin.json: 生成済み
- agents/: 1ファイル
- skills/: 4ファイル
- commands/: 1ファイル
- rule-details: 29ファイルコピー済み

A) 修正依頼 — 構造や設定に変更が必要な場合、内容を指定してください
B) P2（Automated Validation）へ続行 — 3層テストを実行する
```

---

## Output Artifacts

- **plugin.json**: プラグインエントリーポイント定義
- **marketplace.json**: マーケットプレイス公開用メタデータ
- **agents/customer-support-agent.json**: エージェント定義
- **skills/*.json**: フェーズ別skill定義（4ファイル）
- **commands/support.json**: コマンド定義
- **コピー済みrule-detailsツリー**: 29ファイル

---

## Completion Message

```
### REVIEW REQUIRED
[プラグイン構造生成完了]
- 生成ファイル: plugin.json, marketplace.json, agents(1), skills(4), commands(1)
- コピー済み: rule-details 29ファイル
- P2→P1ループ回数: {N}/2

### WHAT'S NEXT
**A) 修正依頼** — プラグイン構造を修正する場合、箇所を指定してください
**B) P2（Automated Validation）へ続行** — 3層テスト（構造+コンテンツ+スモーク）を実行する
```

---

## Error Handling

### エラー1: 29ファイルのうち一部が存在しない

**原因候補**:
- Pitfall 1（ハルシネーション回答）派生: 生成フェーズでファイル出力が省略または中断された
- Pitfall 3（エスカレーション判定ミス）派生: CONDITIONALフェーズのファイル（F1, F2等）が「不要」と誤判断されスキップされた

**対応**:
1. 不足ファイルのリストを明示
2. 該当Phase/Common Rulesの再生成を指示
3. P1はファイルが揃うまで中断（不完全な状態でパッケージングしない）
4. CONDITIONALファイル（R4, R5, F1, F2）は「実行条件付き」であっても定義ファイルは必須であることを確認

### エラー2: P2→P1ループが2回に到達

**原因候補**:
- Pitfall 1（ハルシネーション回答）派生: Phase Rulesの内容がKB（設計書）に基づかない汎用的な記述になっている（Dim 12 FAIL）
- Pitfall 4（トーンミスマッチ）派生: GOOD/BAD例がドメイン固有でなく汎用例になっている（Dim 13 FAIL）

**対応**:
1. `P2_P1_LOOP_LIMIT_REACHED` を記録
2. 失敗内容をユーザーにエスカレーション: 「自動修復の上限に達しました。設計レビューが必要です」
3. P2のFAIL内容を根拠として `core-workflow.md` または対象Phase Rulesの再生成を推奨

---

## GOOD / BAD 具体例

### GOOD 例1: ループカウンター管理

**GOOD**: P2でFAILが返った場合、ループカウンターを確認してから P1 に戻る。カウンター = 2 の場合はループに入らずユーザーにエスカレーション。

### GOOD 例2: 不足ファイル明示

**BAD（NG行動）**: ファイルが27件しかないのに「ほぼ完成」として P2 に進む。

**GOOD**: 不足ファイル名を明示し「follow-up/case-tracking.md が存在しません。F1の再生成が必要です」と通知。29件全揃い後にP2へ。

### BAD 例1: P2スキップ

**BAD（NG行動）**: P1生成後に「時間節約のためP2をスキップ」してリリース。構造テスト未実施のまま運用開始。

**GOOD**: BUILD-TIMEフェーズではP1→P2の順序は必須。P2スキップは許可しない。
