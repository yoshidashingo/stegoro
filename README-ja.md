<p align="center">
  <img src="docs/assets/logo.png" alt="stegoroロゴ" width="160">
</p>

<h1 align="center">
  stegoro
  <br><br>
</h1>

<p align="center">
  目的別AIエージェントを駆動するステアリングポリシーを、構造化された品質保証ワークフローで自動生成します。
</p>

<p align="center">
  <a href="https://github.com/yoshidashingo/steering-policy-generator/blob/main/LICENSE"><img src="https://img.shields.io/github/license/yoshidashingo/steering-policy-generator" alt="license"></a>
  <img src="https://img.shields.io/badge/platform-Claude%20Code-blueviolet" alt="platform">
  <img src="https://img.shields.io/badge/quality-AI--DLC%2011%20dimensions-green" alt="quality">
</p>

<p align="center">
  <a href="README.md">English</a> &bull;
  <a href="README-ja.md">日本語</a>
</p>

## Why — なぜ必要か

目的別AIエージェントの構築は難しい課題です。構造化されたポリシーがなければ、エージェントの出力は一貫性を欠き、重要なバリデーションステップを飛ばし、品質保証が不十分になります。これらのポリシーを手作業で書くのは時間がかかり、ミスが起きやすい作業です。

**stegoro** は、エージェントの目的の理解から完全な品質検証済みポリシーセットの生成まで、ポリシー作成プロセス全体を自動化することでこの問題を解決します。

## What — 何ができるか

[Claude Code](https://claude.ai/code) 上で動作するメタエージェントで、目的別AIエージェント用のステアリングポリシーを生成します。構造化されたマルチフェーズワークフローを通じて、エージェントの振る舞いを統治するルールファイル一式を出力します。

### 特徴

- **4フェーズワークフロー** — Discovery（発見）、Design（設計）、Generation（生成）、Refinement（精製）の4フェーズで網羅的にポリシーを作成
- **適応的深度** — ドメインの複雑さに応じてリサーチ・設計の深度を自動調整（Minimal / Standard / Comprehensive）
- **4種のエージェントタイプ** — Process、Task、Analytical、Hybridの4パターンに対応
- **AI-DLC品質保証** — 適応的ワークフロー、必須チェックポイント、監査証跡、過信防止を含む11品質次元で検証
- **対話的フロー** — 構造化された質問応答と、すべての重要な意思決定ポイントでの承認ゲート
- **セッション継続** — 状態追跡により、セッションをまたいだ中断・再開が可能
- **完全な監査証跡** — ISO 8601タイムスタンプによる全意思決定・ユーザー入力の記録

### 生成される出力

```
.<agent-name>/
├── <agent-name>-rules/
│   └── core-workflow.md              # マスターオーケストレーター
└── <agent-name>-rule-details/
    ├── common/                       # フェーズ横断ルール
    │   ├── welcome-message.md
    │   ├── process-overview.md
    │   ├── question-format-guide.md
    │   ├── content-validation.md
    │   ├── session-continuity.md
    │   ├── error-handling.md
    │   ├── terminology.md
    │   └── ...
    ├── <phase-1>/                    # フェーズ固有ルール
    │   ├── <stage-1>.md
    │   └── ...
    └── <phase-N>/
        └── ...
```

## How — 使い方

### 前提条件

- [Claude Code](https://claude.ai/code) がインストール・設定済みであること

### インストール

マーケットプレイスを追加してプラグインをインストール（初回のみ）:

```bash
/plugin marketplace add yoshidashingo/steering-policy-generator
/plugin install stegoro@steering-policy-generator
```

### 使い方

1. スラッシュコマンドでステアリングポリシーの生成を開始:

```
/stegoro:generate コードレビューエージェント
```

他の例:

```
/stegoro:generate 提案書作成エージェント
/stegoro:generate セキュリティ監査エージェント
```

前回のセッションを再開する場合:

```
/stegoro:generate
```

2. ジェネレーターが4フェーズのワークフローに沿って対話的にガイドします:

| フェーズ | 目的 | 主なアクティビティ |
|---------|------|-----------------|
| **Discovery** | 何を・なぜ作るかを理解 | 目的分析、ドメインリサーチ、利害関係者の特定、スコープ定義 |
| **Design** | ポリシーを設計 | ワークフローアーキテクチャ、共通ルール、フェーズルール、品質メカニズム |
| **Generation** | ポリシーファイルを生成 | コアワークフロー、共通ルール、フェーズルール、統合バリデーション |
| **Refinement** | 品質を保証 | 完全性レビュー、一貫性レビュー、品質キャリブレーション |

3. 各ステージで確認と承認を行ってから次に進みます。明示的な承認なしにジェネレーターが先に進むことはありません。

### このツールで作成されたエージェント例

このリポジトリには、ツールが生成したポリシーの実例が含まれています:

| エージェント | 説明 | ディレクトリ |
|------------|------|------------|
| **Customer Support Agent** | カスタマーサポートチケット対応・品質レビュー | `.customer-support-agent/` |
| **Proposal Writer** | RFP分析・提案書作成 | `.proposal-writer/` |
| **Sales Pipeline Agent** | 巡回サイクル型・営業パイプライン管理 | `.sales-pipeline-agent/` |

## ドキュメント

詳しい使い方は[ドキュメントサイト](https://stegoro.com/)をご覧ください。

## ライセンス

MIT
