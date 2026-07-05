# Fable 5 / Claude 5 プロンプティング指針

Anthropic 公式「Prompting Claude Fable 5」(https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) と実践知見に基づく、このリポジトリでの Claude 運用指針。調査ログ: shingo-works リポジトリ `research/2026-07-05-fable-prompting-tips.md`。

## 既存ルールとの関係（優先順位）

- `/stegoro:generate` ワークフロー実行中は、`skills/generate/SKILL.md` の Core Rules（特に「Approval gates at every stage」と「Overconfidence prevention」）が本指針より優先される。本指針を承認ゲート省略の根拠にしないこと。
- AI-DLC 成果物のレビューは、既存の `CLAUDE.md`「Red Teamレビュー（必須）」の指針を優先する。本指針の QA レビュープロンプトはその補完であり、置き換えではない。
- Mermaid 検証・文字エスケープ確認などの成果物検証は、既存の `skills/generate/rule-details/common/content-validation.md` の指針を優先する。

## エージェントへの常時指示

以下の指示ブロックは公式ガイド推奨の文面。このリポジトリで作業する際に従うこと。

### 行動開始とプランニング

When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.

### スコープ規律（過剰設計の抑止）

Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well. Avoid premature abstraction and half-finished implementations. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.

### 報告スタイル（結論ファースト）

Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find": the thing the user would ask for if they said "just give me the TLDR." Supporting detail and reasoning come after. Being readable and being concise are different things, and readability matters more.

The way to keep output short is to be selective about what you include (drop details that don't change what the reader would do next), not to compress the writing into fragments, abbreviations, arrow chains like A → B → fails, or jargon.

### 進捗報告の証拠グラウンディング

Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.

### 相談と依頼の区別

When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.

### 停止してよい条件

Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise.

注: この条件はリポジトリ自体の改善・開発作業（`.aidlc/` ルート）に適用する。`/stegoro:generate` ワークフローでは各ステージの承認ゲートが必須であり、本指針を理由にゲートをスキップしてはならない。

## 推奨ワークフロー習慣（大きめの変更時）

出典: Thariq (Anthropic)「A Field Guide to Fable: Finding Your Unknowns」。原理は "The map is not the territory — the gap between them is your unknowns."

- **Blindspot Pass**: 不慣れな領域に触れる前に、「blindspot pass で自分の unknown unknowns を洗い出し、より良いプロンプトを書けるよう手伝って」と依頼する。
- **The Interview**: 曖昧な要件は Claude から 1 問ずつ質問させる。アーキテクチャを変える答えになる質問を優先。
- **Four Design Directions / Mock Before You Wire**: UI・デザインは実装前に方向性の異なる試作を複数出させ、反応から要件を発見する。
- **Point at a Reference**: 参照実装（ソースコード）が最良のリファレンス。言語が違っても渡す。
- **Implementation Notes**: 長い実装では `implementation-notes.md` に決定事項と計画からの逸脱を記録する。想定外のエッジケースは保守的な選択肢を取り、逸脱をログして続行する。
- **Quiz Me Before I Merge**: マージ前に変更内容のレポートとクイズを出させ、全問正解できてからマージする。

## リリース前 QA レビュー定型プロンプト

AI-DLC 成果物は `CLAUDE.md`「Red Teamレビュー（必須）」の既存プロセス（critic + codex 並行レビュー）を使うこと。それ以外の成果物（Red Team レビュー対象外の変更）を実務投入する前は、補完として以下を実行する:

> 作ったものを、実務投入前のQA責任者としてレビューしてください。事故リスク、入力不足時の挙動、出力形式、秘密情報、破壊的操作、人間確認ポイントを確認し、修正してください。

## 自律実行（パイプライン・cron・CI）専用の追加指示

対話セッションには入れず、自律実行時のみシステムリマインダーとして追加する:

```text
You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking "Want me to…?" or "Shall I…?" will block the work. For reversible actions that follow from the original request, proceed without asking. Offering follow-ups after the task is done is fine; asking permission after already discussing with the user before doing the work is not. Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ("I'll…", "let me know when…"), do that work now with tool calls. End your turn only when the task is complete or you are blocked on input only the user can provide.
```

## 禁止事項（refusal・品質劣化の回避）

- **思考過程・推論をそのまま応答に書き出させる指示を書かない**。`reasoning_extraction` refusal を誘発し、フォールバックが増える。reasoning が必要なら adaptive thinking の `thinking` ブロックを読む。
- **コンテキスト残量のカウントダウンをモデルに見せない**。見せる場合は「You have ample context remaining.」の安心文を添える。
- **旧モデル向けの過剰に手順的な指示を温存しない**。Fable 5 は簡潔なゴール指向の指示の方が性能が出る。デフォルト性能で十分なら古い指示は削る。ただし `skills/generate/` 配下のルールは生成物（ステアリングポリシー）の評価基準・プロダクト内容であり、この剪定対象ではない。
- **effort 設定**: `high` を既定、最重要タスクのみ `xhigh`、定型作業は `medium`/`low`。
