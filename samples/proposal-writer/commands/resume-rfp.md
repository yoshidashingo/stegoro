---
name: proposal-writer:resume-rfp
description: 中断された提案書作成セッションを再開する
---

## Execution

1. `proposal-docs/` 配下の未完了セッションを一覧表示する
2. ユーザーが再開するRFPを選択する
3. `proposal-docs/<rfp-name>/steering-state.md` を読み込んで進捗を確認する
4. 中断されたフェーズ・ステージから再開する

## Prerequisites

- `proposal-docs/<rfp-name>/steering-state.md` が存在すること

## Output

- 中断点から処理を再開し、前回の成果物を引き継いで継続する
