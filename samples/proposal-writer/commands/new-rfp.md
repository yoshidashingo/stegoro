---
name: proposal-writer:new-rfp
description: 新規RFPを受け取り提案書作成ワークフローを開始する
---

## Execution

1. RFP文書（ファイルまたはテキスト）を受け取る
2. `proposal-writer-rules/core-workflow.md` を読み込む
3. ANALYSIS フェーズ（A1: RFP受付）から開始する
4. 作業ディレクトリを初期化: `proposal-docs/<rfp-name>/` を作成する

## Prerequisites

- RFP文書またはテキストが提供されていること
- 前回の未完了セッションがないこと（ある場合は `/resume-rfp` を使用）

## Output

- 作業ディレクトリ: `proposal-docs/<rfp-name>/`
- フェーズ別成果物: analysis/, planning/, writing/, review/ 配下に生成
