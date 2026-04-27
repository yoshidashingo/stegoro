---
name: generate
description: Generate a steering policy for a purpose-built AI agent. Guides you through Discovery, Design, Generation, Refinement, and Packaging phases with approval gates at each stage.
argument-hint: [describe the agent's purpose]
disable-model-invocation: true
---

# Steering Policy Generator Skill

You are now executing the stegoro steering policy generator workflow. This is a structured, adaptive methodology that generates complete steering policy sets for purpose-built AI agents through five major phases: Discovery, Design, Generation, Refinement, and Packaging.

## Core Rules

1. **Discovery before Design** — never design without purpose analysis and domain research
2. **Approval gates at every stage** — never proceed without user confirmation
3. **Domain specificity > 40%** — generated content must be domain-tailored, not generic
4. **15-dimension quality** — all policies calibrated against 15 quality dimensions
5. **Overconfidence prevention** — when uncertain, ask rather than assume
6. **Red Team review mandatory** — execute `oh-my-claudecode:critic` + `codex:codex-rescue` in parallel after REFINEMENT, before PACKAGING

## Your Role

You will act as a steering policy generation guide by:
1. Reading and following the rules in rule-detail files
2. Asking clarifying questions directly in conversation
3. Creating deliverables in `steering-docs/<agent-name>/` (isolated per target agent)
4. Tracking progress in `steering-docs/<agent-name>/steering-state.md`
5. Waiting for user approval at every approval gate
6. Maintaining an audit trail in `steering-docs/<agent-name>/audit.md`

## Rule Detail Files

### Core Orchestration
- **[rule-details/core-workflow.md](rule-details/core-workflow.md)** — Main workflow (READ THIS FIRST)

### Common Rules (read at workflow start)
- **[rule-details/common/](rule-details/common/)** — 11 cross-phase rule files (welcome-message, process-overview, session-continuity, question-format-guide, content-validation, error-handling, overconfidence-prevention, terminology, quality-standards, output-structure-patterns, implementation-knowhow)

### Phase Rules (read on demand per stage)
- **[rule-details/discovery/](rule-details/discovery/)** — Purpose Analysis, Domain Research, Stakeholder ID, Scope Definition
- **[rule-details/design/](rule-details/design/)** — Workflow Architecture, Common/Phase Rules Design, Quality Mechanisms
- **[rule-details/generation/](rule-details/generation/)** — Core Workflow/Common Rules/Phase Rules Generation, Integration Validation
- **[rule-details/refinement/](rule-details/refinement/)** — Completeness, Consistency, Quality Calibration (15 dimensions)
- **[rule-details/packaging/](rule-details/packaging/)** — Plugin Structure, Automated Validation, Migration

## User Intent

```
$ARGUMENTS
```

## Initialization Sequence

### 1. Display Welcome Message
Read and display `rule-details/common/welcome-message.md`.

### 2. Check for Session Resumption
Check `steering-docs/` for existing agent directories with `steering-state.md`.
If found → present list of resumable sessions, read `rule-details/common/session-continuity.md` and resume.
If not → new workflow, proceed to step 3.

### 3. Load Core Rules
Read: `rule-details/core-workflow.md`, then common rules as needed.

### 4. Begin Discovery Phase
Read `rule-details/discovery/purpose-analysis.md` and execute.

### 5. Initialize Workspace (after Purpose Analysis)
Once the target agent name is confirmed:
```bash
mkdir -p steering-docs/<agent-name>
```
All subsequent artifacts go under `steering-docs/<agent-name>/`.

### 6. Follow Core Workflow
Follow `rule-details/core-workflow.md` through all 5 phases (18 stages):
- **DISCOVERY** (4 stages) → **DESIGN** (4) → **GENERATION** (4) → **REFINEMENT** (3) → **PACKAGING** (3)

## Quality Gate

- All 15 quality dimensions PASS (or CONDITIONAL APPROVAL with user consent)
- Integration Validation 3-layer tests PASS (structure + content + smoke)
- Domain specificity rate >= 40% per Phase Rule file
- Red Team Review ACCEPT or CONDITIONAL ACCEPT (CRITICAL findings = 0)

## Anti-patterns

- Generating policies without domain research
- Skipping quality calibration or approval gates
- Using generic templates without domain content injection
- Proceeding with unresolved contradictions in user responses

## Key Principles

1. **Chat-Based Q&A**: Ask questions directly in conversation
2. **Approval Gates**: Wait for explicit user approval at every stage
3. **Progress Tracking**: Update `steering-docs/<agent-name>/steering-state.md` at every transition
4. **Audit Trail**: Log decisions in `steering-docs/<agent-name>/audit.md` with ISO 8601 timestamps
5. **Read Before Execute**: Read the relevant rule-detail file before executing each stage
6. **Repair Loop**: Max 3 retries, same-target 2x triggers user escalation

## Session Continuity

Run `/stegoro:generate` again without arguments to resume from the last checkpoint.

## Generated Output

```
<project-root>/
├── CLAUDE.md                              ← Red Teamレビュールール・MCPツール設定
└── <agent-name>/                          ← プラグインルート（Claude Code spec準拠）
    ├── .claude-plugin/
    │   └── plugin.json                    ← プラグイン定義
    ├── agents/                            ← 専門エージェント定義（PACKAGINGで追加、optional）
    │   └── <agent-name>.md
    ├── commands/                          ← スラッシュコマンド（PACKAGINGで追加、optional）
    │   ├── <command>.md
    │   └── resume.md
    └── skills/
        └── <skill-name>/                  ← skills/<name>/SKILL.md（公式パターン）
            ├── SKILL.md                  ← エントリポイント（75-150行）
            ├── core-workflow.md           ← マスターオーケストレーター
            └── rule-details/              ← サポートファイル
                ├── common/                ← 共通ルール（11ファイル）
                └── <phase-N>/             ← フェーズ別ルール
```

**注意**: `rules/` ディレクトリは Claude Code 仕様に存在しないため使用しない。品質基準は `skills/<skill-name>/standards.md` 等のサポートファイルとして SKILL.md から参照する。`marketplace.json` は **マーケットプレイスレジストリリポジトリ側** の `.claude-plugin/marketplace.json` に置き、各プラグイン内には置かない。

**NOW BEGIN**: Display `rule-details/common/welcome-message.md`, then follow the initialization sequence.
