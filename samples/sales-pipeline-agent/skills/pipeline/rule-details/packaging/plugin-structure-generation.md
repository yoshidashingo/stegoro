# P1: Plugin Structure Generation — Phase Rule

## Purpose

検証済みのポリシーファイル群をClaude Codeプラグイン形式にパッケージングする。`plugin.json`・`marketplace.json`・エージェント定義・スキル定義・コマンド定義を生成し、ディレクトリ構造を構築する。参照パスの整合性チェックと全ポリシーファイルの存在確認を実施する。このフェーズはポリシー初期構築・更新時のみ実行し、巡回サイクルのランタイムフローには含まれない。

**Classification**: ALWAYS EXECUTE（ビルドタイム実行）
**Approval Gate**: あり（CP-8: プラグイン構造・参照整合性の確認）
**参照元**: `workflow-architecture.md` Build-Time: PACKAGING セクション

---

## Prerequisites

- 全18ファイルのPhase Rulesが確定していること（surveillance/4 + assessment/4 + dispatch/5 + reporting/3 + packaging/2）
- Common Rulesが確定していること（`common/` ディレクトリ）
- `.sales-pipeline-agent/` ディレクトリへの書き込み権限があること
- `plugin.json` スキーマ定義（Claude Codeプラグイン仕様）が参照可能であること

---

## Execution Classification

**ALWAYS EXECUTE**（ビルドタイム）

- Execute IF: ポリシー初期構築時、またはポリシーファイルの更新時
- Skip IF: ランタイム巡回サイクル中（このフェーズは巡回フローに含まれない）

---

## Execution Steps

### Step 1: 全ポリシーファイルの存在確認

パッケージング対象の全ファイルが存在することを確認する。

**確認対象ファイルリスト**:

```
rule-details/
├── surveillance/
│   ├── schedule-check.md
│   ├── source-connection.md
│   ├── data-collection.md
│   └── change-detection.md
├── assessment/
│   ├── deal-health-scoring.md
│   ├── action-identification.md
│   ├── priority-ranking.md
│   └── authority-classification.md
├── dispatch/
│   ├── auto-execute.md
│   ├── approval-request.md
│   ├── approval-processing.md
│   ├── loop-control.md
│   └── rollback-check.md
├── reporting/
│   ├── daily-summary-generation.md
│   ├── next-day-prep.md
│   └── audit-log-finalization.md
└── packaging/
    ├── plugin-structure-generation.md
    └── automated-validation.md

./
├── core-workflow.md
├── action-authority.md
└── common/ （共通ルール群）
```

**ファイル不在の場合**: パッケージングを停止し、欠損ファイルリストを報告する。

### Step 2: plugin.json の生成

Claude Codeプラグイン仕様に従って `plugin.json` を生成する。

```json
{
  "name": "sales-pipeline-agent",
  "version": "1.0.0",
  "display_name": "営業パイプライン管理エージェント",
  "description": "Salesforce・Gmail・Google Calendar・Slackの4ソースを定期巡回し、対応が必要な案件を検出・自動対応・承認制管理を行うHybrid型自律エージェント",
  "type": "agent",
  "entry_point": "agents/sales-pipeline-agent.md",
  "rules": [
    "./core-workflow.md",
    "./action-authority.md"
  ],
  "rule_details": "rule-details/",
  "config": "config/",
  "state": "state/",
  "logs": "logs/",
  "skills": [
    "skills/pipeline-surveillance.md",
    "skills/deal-health-scoring.md"
  ],
  "commands": [
    "commands/start-cycle.md",
    "commands/check-status.md",
    "commands/manual-trigger.md"
  ],
  "permissions": {
    "salesforce": ["read", "write", "bulk_api"],
    "gmail": ["read", "send"],
    "google_calendar": ["read"],
    "slack": ["read", "write", "dm"]
  },
  "schedule": {
    "daytime": "0 * * * 1-5",
    "nighttime_check": "0 20 * * 1-5"
  }
}
```

### Step 3: エージェント定義ファイルの生成

`agents/sales-pipeline-agent.md` を生成する。

**エージェント定義テンプレート**:
```markdown
# sales-pipeline-agent

## Agent Overview
営業パイプライン管理エージェント。4データソースを定期巡回し、
対応要件を持つ案件を検出・分類・実行する自律型Hybridエージェント。

## Operational Flow
workflow-architecture.md のフローに従い、以下の4フェーズを実行:
1. SURVEILLANCE（S1→S2→S3→S4）
2. ASSESSMENT（A1→A2→A3→A4）
3. DISPATCH（D1→D2→D3→D4→D5）
4. REPORTING（R1→R2→R3）

## Phase Rules Reference
各ステージの詳細は rule-details/ を参照。

## Configuration
設定ファイルは config/ ディレクトリを参照。
```

### Step 4: ディレクトリ構造の生成

必要なディレクトリとプレースホルダーファイルを生成する。

**生成するディレクトリ**:
```
.sales-pipeline-agent/
├── agents/                    # エージェント定義
├── skills/                    # スキル定義
├── commands/                  # コマンド定義
├── config/                    # 設定ファイル
│   ├── stage-thresholds.yaml.example
│   ├── priority-config.yaml.example
│   ├── action-authority.md.example
│   ├── notification-settings.yaml.example
│   ├── data-source-config.md.example
│   └── japan-holidays.json.example
├── state/                     # 実行時状態ファイル（.gitignore対象）
├── logs/                      # 監査ログ（.gitignore対象）
└── rule-details/  # Phase Rules（既存）
```

**設定ファイルのexampleテンプレート生成**:
- 設定値のサンプル値と説明コメントを含む `.example` ファイルを生成
- 実際の設定値（OAuthトークン等）は含まない

### Step 5: 参照パスの整合性チェック

生成した全ファイル間の参照パスが正しいことを確認する。

**チェック項目**:
- [ ] `plugin.json` の全参照先ファイルが存在するか
- [ ] Phase Rules内の `config/` 参照パスが `config/` ディレクトリに実在するか（またはexampleが存在するか）
- [ ] `workflow-architecture.md` のフロー図と Phase Rules のステージ名が一致しているか
- [ ] `action-authority.md` への参照がA4・D1・D3で正しく記載されているか

**整合性エラーの報告**:
```yaml
integrity_check_result:
  total_references_checked: 47
  broken_references: 0
  missing_files: 0
  stage_name_mismatches: 0
  status: "PASS"
```

---

## Output Artifacts

P1 が生成するアーティファクト:

1. **`plugin.json`** — Claude Codeプラグインのエントリポイント
2. **`agents/sales-pipeline-agent.md`** — エージェント定義
3. **ディレクトリ構造**（`config/`・`state/`・`logs/`・`skills/`・`commands/`）
4. **設定ファイルテンプレート**（`.example` ファイル群）
5. **整合性チェック結果** — P2の検証入力に使用

---

## Completion Message

P1 完了時のメッセージ形式は以下の通り。

```
### REVIEW REQUIRED — P1: Plugin Structure Generation 完了

生成結果:
- plugin.json: 生成済み
- エージェント定義: agents/sales-pipeline-agent.md
- ディレクトリ構造: [X]ディレクトリ生成
- 設定テンプレート: [X]ファイル生成

CP-8チェック:
- ファイル存在確認: [X]件 / [X]件 PASS
- 参照パス整合性: [PASS / 要修正 X件]
- ステージ名整合性: [PASS / 要修正 X件]

### WHAT'S NEXT
**A) 修正依頼** — 構造やパス参照の問題を修正する
**B) [P2 Automated Validation]へ続行** — 3層テスト検証へ進む
```

---

## Error Handling

### エラー 1: ポリシーファイル不足でのパッケージング強行

**症状**: 18ファイルのうち2ファイルが未生成の状態でパッケージングを実行し、不完全なプラグインが生成される。ランタイムで参照エラーが発生する。
**原因**: Step 1のファイル存在確認が省略されている。
**対処**: 全18ファイルの存在確認が通過しない限りStep 2以降を実行しない。欠損ファイルリストを明示して担当者に生成を促す。

**GOOD例**:
```
Step 1: ファイル存在確認
→ surveillance/change-detection.md: 存在 ✓
→ reporting/audit-log-finalization.md: 存在なし ✗
→ packaging/automated-validation.md: 存在なし ✗
→ パッケージング停止
→ 報告: 「2ファイルが不足しています。生成後に再実行してください」
```

**BAD例**:
```
Step 1: スキップ（全ファイル存在を前提）
→ plugin.json に audit-log-finalization.md への参照を記載
→ P2検証: ファイル参照エラーで FAIL
→ ランタイムで R3実行時に参照エラー
→ サイクルが R3で常に失敗する状態に
```

### エラー 2: 設定ファイルの機密情報の混入（Pitfall 4対策）

**症状**: OAuthトークン・APIキーを含む実際の設定ファイルを `.example` ファイルに誤って含めてしまい、設定ファイルがリポジトリにコミットされて機密情報が漏洩する。トークン漏洩はSalesforce APIガバナーリミットの不正消費（Pitfall 4）や、Gmailへの不正アクセスにつながる。
**原因**: `config/` の実際の設定ファイルと `.example` テンプレートを混同した。
**対処**: `.example` ファイルには必ずプレースホルダー値のみを使用する。実際の値が含まれていないかをステップ完了前に確認する。`state/`・`logs/`・`config/`（実設定）を `.gitignore` に必ず追加する。OAuthトークン管理はSOC 2データセキュリティ基準（Domain Research Quality Standards参照）に準拠する。

**GOOD例**:
```
config/oauth-tokens.json.example:
{
  "salesforce_access_token": "YOUR_SALESFORCE_ACCESS_TOKEN",
  "gmail_refresh_token": "YOUR_GMAIL_REFRESH_TOKEN"
}
→ プレースホルダー値のみ（実トークンなし）
→ .gitignore: config/oauth-tokens.json（実ファイルは除外）
→ Salesforce APIの不正利用・ガバナーリミット超過リスクを排除（Pitfall 4対策）
```

**BAD例**:
```
config/oauth-tokens.json.example:
{
  "salesforce_access_token": "00D5g000005kHJs!AQEAQF...",  # 実際のトークン
  "gmail_refresh_token": "1//0ghY..."
}
→ 機密情報がexampleファイルに混入
→ リポジトリにコミットして漏洩リスク
→ 第三者がSalesforce APIを無制限に呼び出し、ガバナーリミットを枯渇させる可能性
（Pitfall 4: API制限への対応不足と同等のリスク）
```
