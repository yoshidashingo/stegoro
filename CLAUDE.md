# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

目的別エージェント用のステアリングポリシー作成器。特定業務の目的別エージェントを駆動するステアリングポリシーを自動生成する。

## ルーティングルール（必須）

このリポジトリには2つのルールセットがあり、作業コンテキストに応じて使い分ける。

### リポジトリ自体の改善・開発時 → `.aidlc/` ルールを使用

- **対象**: このリポジトリのコード・構造・ドキュメントの改善、新機能追加、リファクタリング、バグ修正
- **ルール読み込み**: `.aidlc/aws-aidlc-rules/core-workflow.md` をエントリポイントとして読み込む
- **詳細ルール**: `.aidlc/aws-aidlc-rule-details/` 配下のフェーズ別ルールに従う
- **フェーズ**: Inception → Construction → Operations

### ステアリングポリシーの生成時 → `skills/generate/` を使用

- **対象**: `/stegoro:generate` スキル経由、または「XXX業務のステアリングポリシーを生成してください」等のポリシー生成依頼
- **エントリポイント**: `skills/generate/SKILL.md`
- **ルール詳細**: `skills/generate/rule-details/` 配下のフェーズ別ルールに従う
- **フェーズ**: Discovery → Design → Generation → Refinement

### 判定基準

| ユーザーの依頼 | 使用ルール |
|---|---|
| `/stegoro:generate` | `skills/generate/` |
| 「ステアリングポリシーを生成して」 | `skills/generate/` |
| 「XXX業務のエージェントを作って」 | `skills/generate/` |
| 「このリポジトリを改善して」 | `.aidlc/` |
| 「バグを修正して」 | `.aidlc/` |
| 「新機能を追加して」 | `.aidlc/` |
| 「ドキュメントを更新して」 | `.aidlc/` |

## ディレクトリ構造

```text
.aidlc/                          # リポジトリ改善用ルール（AI-DLC）
├── aws-aidlc-rules/
│   └── core-workflow.md         # メインワークフロー
└── aws-aidlc-rule-details/
    ├── common/                  # 共通ルール
    ├── inception/               # 企画フェーズ
    ├── construction/            # 構築フェーズ
    └── operations/              # 運用フェーズ

.claude-plugin/                  # Claude Codeプラグインメタデータ
├── plugin.json                  # プラグイン定義
└── marketplace.json             # マーケットプレイス登録情報

skills/                          # Claude Codeスキル
└── generate/                    # ステアリングポリシー生成スキル
    ├── SKILL.md                 # スキルエントリポイント
    └── rule-details/            # ルール詳細
        ├── core-workflow.md     # メインワークフロー
        ├── common/              # 共通ルール
        ├── discovery/           # 調査フェーズ
        ├── design/              # 設計フェーズ
        ├── generation/          # 生成フェーズ
        └── refinement/          # 精緻化フェーズ

steering-docs/                   # ポリシー生成時の作業ドキュメント
.claude/rules/                   # Claude Code ルール（ルーティング、プロンプティング指針）
```

## Red Teamレビュー（必須）

AI-DLCワークフローでレビューが必要な成果物を作成・改善した場合、**必ず** Red Teamレビューを実施すること。

### 対象

以下の成果物が作成または大幅改訂された場合:
- DISCOVERY成果物（purpose-analysis-summary, domain-research-summary, scope-definition-summary）
- DESIGN成果物（workflow-architecture, common-rules-design, phase-rules-design, quality-mechanisms-design）
- GENERATION成果物（core-workflow.md, common rules, phase rules）
- REFINEMENT成果物（completeness-report, consistency-report, calibration-scorecard）

### 実施方法

2つのレビューエージェントを**並行実行**する:

1. **`oh-my-claudecode:critic`** — 厳格なRed Teamレビュー。REJECT/ACCEPTの判定、CRITICAL/MAJOR/MINOR finding、具体的改善提案
2. **`codex:codex-rescue`** — 定量分析・代替実装の提案。ファイル別品質スコア、具体的な改善コード例

### レビュー基準

- **ACCEPT**: CRITICAL finding なし、MAJOR finding 2件以下
- **CONDITIONAL ACCEPT**: CRITICAL なし、MAJOR 3件以上 → 指摘対応後に再レビュー不要
- **REJECT**: CRITICAL 1件以上 → 指摘を反映して改訂後、再レビュー必須

### ワークフロー

```
成果物作成 → Red Teamレビュー（critic + codex並行）→ 指摘反映 → [REJECTの場合: 再レビュー] → ユーザー承認
```

### 例外

- 既存ファイルの軽微な修正（typo修正、フォーマット統一等）はレビュー不要
- ユーザーが明示的にレビュースキップを指示した場合

## プロンプティング指針

Claude (Fable 5) での作業スタイルは以下の指針に従うこと。既存のルーティングルール・承認ゲート・Red Teamレビューが優先される。

@.claude/rules/fable-prompting.md

## 言語

日本語で応答してください。
