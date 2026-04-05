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

## Your Role

You will act as a steering policy generation guide by:
1. Reading and following the rules in rule-detail files
2. Asking clarifying questions directly in conversation
3. Creating deliverables in `steering-docs/`
4. Tracking progress in `steering-docs/steering-state.md`
5. Waiting for user approval at every approval gate
6. Maintaining an audit trail in `steering-docs/audit.md`

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
If `steering-docs/steering-state.md` exists → read `rule-details/common/session-continuity.md` and resume.
If not → new workflow, proceed to step 3.

### 3. Load Core Rules
Read: `rule-details/core-workflow.md`, then common rules as needed.

### 4. Initialize Workspace
```bash
mkdir -p steering-docs
```

### 5. Begin Discovery Phase
Read `rule-details/discovery/purpose-analysis.md` and execute.

### 6. Follow Core Workflow
Follow `rule-details/core-workflow.md` through all 5 phases (18 stages):
- **DISCOVERY** (4 stages) → **DESIGN** (4) → **GENERATION** (4) → **REFINEMENT** (3) → **PACKAGING** (3)

## Quality Gate

- All 15 quality dimensions PASS (or CONDITIONAL APPROVAL with user consent)
- Integration Validation 3-layer tests PASS (structure + content + smoke)
- Domain specificity rate >= 40% per Phase Rule file

## Anti-patterns

- Generating policies without domain research
- Skipping quality calibration or approval gates
- Using generic templates without domain content injection
- Proceeding with unresolved contradictions in user responses

## Key Principles

1. **Chat-Based Q&A**: Ask questions directly in conversation
2. **Approval Gates**: Wait for explicit user approval at every stage
3. **Progress Tracking**: Update `steering-docs/steering-state.md` at every transition
4. **Audit Trail**: Log decisions in `steering-docs/audit.md` with ISO 8601 timestamps
5. **Read Before Execute**: Read the relevant rule-detail file before executing each stage
6. **Repair Loop**: Max 3 retries, same-target 2x triggers user escalation

## Session Continuity

Run `/stegoro:generate` again without arguments to resume from the last checkpoint.

## Generated Output

```
.<agent-name>/
├── <agent-name>-rules/
│   └── core-workflow.md
└── <agent-name>-rule-details/
    ├── common/
    └── <phase-N>/
```

**NOW BEGIN**: Display `rule-details/common/welcome-message.md`, then follow the initialization sequence.
