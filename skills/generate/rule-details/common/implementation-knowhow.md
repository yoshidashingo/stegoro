# 実装ノウハウ — ステアリングポリシー生成

## ディレクトリ構造の統一パターン

### 標準構造（GENERATIONで作成 → PACKAGINGで完成）

すべての生成ファイルは `<agent-name>/` ディレクトリ（ドットなし）に格納する。

```text
<agent-name>/
├── .claude-plugin/
│   └── plugin.json                        ← プラグイン定義（PACKAGINGフェーズで追加）
├── agents/                                ← オプション（PACKAGINGフェーズで追加）
├── commands/                              ← オプション（PACKAGINGフェーズで追加）
└── skills/
    └── <skill-name>/                      ← スキル本体
        ├── SKILL.md                       ← スキルエントリポイント（PACKAGINGで生成）
        ├── core-workflow.md               ← マスターオーケストレーター（GENERATIONで生成）
        ├── standards.md                   ← 品質基準（オプション、SKILL.mdから参照）
        └── rule-details/                  ← ルール詳細（GENERATIONで生成）
            ├── common/
            ├── <phase-1>/
            └── <phase-N>/
```

**重要**:
- `marketplace.json` は per-plugin では作らない（公式仕様外）
- `rules/` ディレクトリは公式仕様に存在しない（品質基準は `skills/<skill-name>/` 配下に置く）
- `<agent-name>-rules/` `<agent-name>-rule-details/` のような top-level ディレクトリは作らず、すべて `skills/<skill-name>/` 配下に統合する

---

## スキル設計10パターン（Domain Research由来）

高品質なClaude Codeスキルに共通する設計パターン:

| # | パターン | 内容 | 適用先 |
|---|---------|------|--------|
| 1 | 明示的アクティベーショントリガー | ユーザーがどう呼び出すか明確に定義 | SKILL.md冒頭 |
| 2 | 4-5個の記憶しやすいコアルール | 中核的な行動指針を簡潔に列挙 | SKILL.md, welcome-message |
| 3 | Quality Gate | 品質基準の明示と自動チェック | quality-standards, SKILL.md |
| 4 | 番号付きワークフロー | フェーズ・ステージに番号を振り順序を明示 | core-workflow, process-overview |
| 5 | アンチパターンセクション | やってはいけないことを明示 | overconfidence-prevention |
| 6 | コンテンツ密度制限 | ファイル種別ごとの行数ガイドライン | content-validation |
| 7 | クロススキル参照 | 関連スキルへの参照を明示 | implementation-knowhow |
| 8 | ツール/MCP連携の明示 | 使用するMCPツールとフォールバックを定義 | domain-research, SKILL.md |
| 9 | グレースフルデグラデーション | ツール不在時のフォールバック手順 | error-handling |
| 10 | エビデンスファースト | スコアカードにEvidence列を必須化 | quality-standards |

---

## プラグイン構造ガイド

### plugin.json の必須フィールド

```json
{
  "name": "<agent-name>",
  "version": "0.1.0",
  "description": "..."
}
```

**注意（公式仕様準拠）**:
- 必須フィールドは `name` のみ（`version`/`description` は推奨）
- `agents`/`skills`/`commands`/`hooks`/`mcpServers`/`lspServers` は **カスタムパスを使うときだけ** 宣言。空配列はデフォルト自動探索を SUPPRESS するため避ける
- `rules` フィールドは **Claude Code 公式仕様に存在しない**。品質基準は `skills/<skill-name>/` 配下のサポートファイル（例: `standards.md`）に格納し SKILL.md から参照すること

### marketplace.json は per-plugin で作らない

公式仕様では `marketplace.json` は **マーケットプレイスレジストリリポジトリの root** に置く1ファイルであり、各プラグインの `.claude-plugin/` 内には置かない。プラグイン公開時は、レジストリ側 `.claude-plugin/marketplace.json` の `plugins[]` 配列に以下のようなエントリを追加する:

```json
{
  "name": "<agent-name>",
  "source": "./<agent-name>",
  "description": "...",
  "version": "0.1.0",
  "category": "...",
  "tags": ["steering-policy", "..."]
}
```

### SKILL.md 設計ガイド（75-150行）

SKILL.mdは簡潔なエントリポイント。詳細はrule-detailsに委譲する。

**必須セクション**:
1. **Activation Trigger** — いつこのスキルが呼び出されるか
2. **Core Rules** — 4-5個の中核ルール
3. **Workflow Summary** — フェーズ/ステージの概要
4. **Quality Gate** — 最低品質基準
5. **References** — rule-detailsへのパス

**アンチパターン**:
- SKILL.mdに全ワークフロー詳細を書く（→ rule-detailsに委譲）
- 150行を超える（→ 分割が必要）
- rule-detailsへの参照がない（→ 実行時にルールが読み込まれない）

### エージェント定義設計

エージェントは責務で分割する:

| エージェント | 責務 | 使用フェーズ |
|------------|------|-----------|
| Orchestrator | ワークフロー全体の調整、状態管理 | 全フェーズ |
| Domain Researcher | ドメイン調査、ベストプラクティス収集 | DISCOVERY |
| Policy Generator | ポリシーファイル生成 | GENERATION |
| Quality Reviewer | 品質検証、スコアリング | REFINEMENT |

### コマンド定義パターン

```markdown
---
name: <prefix>:<command-name>
description: ...
---

# Command: <command-name>

## Execution
1. [手順]
2. [手順]
```

典型的なコマンド:
- `new-policy` — 新規ポリシー生成開始
- `resume-policy` — 中断されたポリシー生成を再開
- `validate-policy` — 既存ポリシーの品質検証

---

## マイグレーションガイド

### レガシー構造 → プラグイン構造

1. GENERATION/REFINEMENTで改善済みファイルを `<agent-name>-rule-details/` にコピー
2. プラグイン固有ファイル（plugin.json, agents/, skills/, commands/）を新規生成
3. CLAUDE.md等の参照パスを更新
4. レガシーディレクトリに `DEPRECATED` 注記を追加（**削除はしない**）

### パスマッピング例

| 旧パス | 新パス |
|--------|--------|
| `.steering/aws-aidlc-rules/core-workflow.md` | `<agent>/skills/<skill-name>/core-workflow.md` |
| `.steering/aws-aidlc-rule-details/common/` | `<agent>/skills/<skill-name>/rule-details/common/` |
| `.steering/aws-aidlc-rule-details/<phase>/` | `<agent>/skills/<skill-name>/rule-details/<phase>/` |

---

## ポリシー生成時の教訓

### Discovery フェーズが最重要

- Purpose Analysis で対象エージェントの本質を正確に捉えることが全体品質を決める
- ドメインリサーチは省略せず、最低でも Standard 深度で実施する
- MCP連携: Context7→公式ドキュメント、Exa→Web調査、gh search→実装例
- ツール不在時はユーザーへの直接質問にフォールバック

### Generation フェーズでの注意点

- core-workflow.md 内のすべてのファイル参照が実ファイルに解決されることを検証する
- 共通ルールファイルは対象ドメインに適応させてから生成する（テンプレートのコピーではない）
- ドメインコンテンツ注入ヒューリスティクス（G3の5ステップ）を必ず実行する
- ドメイン特化率40%未満のファイルは追加注入対象としてフラグする

### 並行生成パターン（効率化オプション）

DESIGN成果物が全てApproved済みの場合、GENERATIONの3ステージ（G1/G2/G3）は並行実行が可能:

| Agent | Input | Output |
|-------|-------|--------|
| G1: Core Workflow | Workflow Architecture + Quality Mechanisms | core-workflow.md |
| G2: Common Rules | Common Rules Design + Domain Research | common/*.md |
| G3: Phase Rules | Phase Rules Design + Domain Research | phase-N/*.md |

**前提条件**: 全DESIGN成果物がApproved済み
**リスク**: 並行生成後のG4(Integration Validation)で不整合検出時の修正コスト増
**推奨**: 大規模ポリシー（25+ファイル）でコンテキスト節約が必要な場合のオプション

### Refinement フェーズでの注意点

- 15品質次元すべてを定量的にエビデンス付きで評価する
- 修復判断ツリーに従い、FAIL時の戻り先を自動ルーティングする
- 修復ループは最大3回。同一ターゲット2回でユーザーエスカレーション

## パス参照のルール

### core-workflow.md 内の参照

- `rule-details/` ディレクトリ内の相対パスで参照する
- 例: `common/welcome-message.md`, `discovery/purpose-analysis.md`

### ルーティングとCLAUDE.mdの連携

- リポジトリ改善 → `.aidlc/` ルールで駆動
- ポリシー生成 → `skills/generate/` ルールで駆動
- CLAUDE.md に判定テーブルを記載し、依頼内容に応じて自動選択
