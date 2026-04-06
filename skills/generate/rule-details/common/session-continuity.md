# Session Continuity Templates

## Welcome Back Prompt Template
When a user returns to continue work on an existing steering policy creation project, present this prompt:

```markdown
**Welcome back! I can see you have an existing Steering Policy creation project in progress.**

Based on your steering-state.md, here's your current status:
- **Target Agent**: [agent-name]
- **Agent Type**: [Process/Task/Analytical/Hybrid]
- **Current Phase**: [DISCOVERY/DESIGN/GENERATION/REFINEMENT/PACKAGING]
- **Current Stage**: [Stage Name]
- **Last Completed**: [Last completed step]
- **Next Step**: [Next step to work on]

**What would you like to work on today?**

A) Continue where you left off ([Next step description])
B) Review a previous stage ([Show available stages])

[Answer]:
```

## MANDATORY: Working Directory Isolation

Each target agent's working artifacts are stored in an isolated directory:
- **Working directory**: `steering-docs/<agent-name>/`
- **State file**: `steering-docs/<agent-name>/steering-state.md`
- **Audit file**: `steering-docs/<agent-name>/audit.md`
- **Phase artifacts**: `steering-docs/<agent-name>/<phase-name>/`

When resuming, scan `steering-docs/` for subdirectories containing `steering-state.md` to discover resumable sessions.

## MANDATORY: Session Continuity Instructions

1. **Scan for existing sessions** — look for `steering-docs/*/steering-state.md` files. If multiple found, present list and let user choose.
2. **Parse current status** from the state file to populate the prompt
3. **MANDATORY: Load Previous Stage Artifacts** — Before resuming any stage, automatically read all relevant artifacts from previous stages:
   - **Purpose Analysis**: Read purpose-analysis results, agent classification
   - **Domain Research**: Read domain findings, best practices, pitfalls
   - **Stakeholder Identification**: Read stakeholder map (if executed)
   - **Scope Definition**: Read scope boundaries, estimates, directory structure
   - **Workflow Architecture**: Read phase/stage design, workflow diagram
   - **Common Rules Design**: Read common rules inventory, adaptation notes
   - **Phase Rules Design**: Read rule file designs, content outlines
   - **Quality Mechanisms Design**: Read quality plan, checkpoint design
   - **Generation Stages**: Read all generated files relevant to current context
   - **Refinement Stages**: Read all reports, calibration results
4. **Smart Context Loading by Phase**:
   - **DISCOVERY Phase**: Load any prior DISCOVERY artifacts
   - **DESIGN Phase**: Load all DISCOVERY artifacts + prior DESIGN artifacts
   - **GENERATION Phase**: Load all DISCOVERY + DESIGN artifacts + prior GENERATION artifacts
   - **REFINEMENT Phase**: Load ALL artifacts from all phases
   - **PACKAGING Phase**: Load ALL artifacts + plugin design from Scope Definition
   - **Repair Loop Resume**: Load Repair Loop History from steering-state.md, identify current loop count and last repair target
5. **Adapt options** based on agent type classification and current phase
6. **Show specific next steps** rather than generic descriptions
7. **Log the continuity prompt** in audit.md with timestamp
8. **Context Summary**: After loading artifacts, provide brief summary of what was loaded for user awareness
9. **Asking questions**: ALWAYS ask clarification or user feedback questions by placing them in .md files. DO NOT place the multiple-choice questions in-line in the chat session.

## State File Format

### steering-state.md Structure

```markdown
# Steering Policy Maker — State

## Project Information
- **Target Agent**: [agent-name]
- **Agent Type**: [Process/Task/Analytical/Hybrid]
- **Domain**: [domain description]
- **Created**: [ISO timestamp]
- **Last Updated**: [ISO timestamp]

## Phase Progress

### DISCOVERY PHASE
- [x] Purpose Analysis — COMPLETED [timestamp]
- [x] Domain Research — COMPLETED [timestamp]
- [ ] Stakeholder Identification — SKIPPED (single user type)
- [x] Scope Definition — COMPLETED [timestamp]

### DESIGN PHASE
- [x] Workflow Architecture — COMPLETED [timestamp]
- [ ] Common Rules Design — IN PROGRESS
- [ ] Phase Rules Design — PENDING
- [ ] Quality Mechanisms Design — PENDING

### GENERATION PHASE
- [ ] Core Workflow Generation — PENDING
- [ ] Common Rules Generation — PENDING
- [ ] Phase Rules Generation — PENDING
- [ ] Integration Validation — PENDING

### REFINEMENT PHASE
- [ ] Completeness Review — PENDING
- [ ] Consistency Review — PENDING
- [ ] Quality Calibration — PENDING

### PACKAGING PHASE
- [ ] Plugin Structure Generation — PENDING
- [ ] Automated Validation — PENDING
- [ ] Migration Execution — PENDING / SKIPPED (reason)

## Repair Loop History
- [timestamp] Loop #[N]: [Source Stage] → [Target Stage]: [Reason] (dimension: Dim [N])

## Current Position
- **Phase**: DESIGN
- **Stage**: Common Rules Design
- **Step**: Step 3 — Identify domain-specific common rules
- **Status**: Awaiting user response to common-rules-design-questions.md

## Key Decisions
- [timestamp] Agent classified as Process Agent
- [timestamp] Domain: Software Development — Code Review
- [timestamp] Scope: 3 phases, 8 stages, ~20 rule files
- [timestamp] Workflow architecture approved with 3-phase structure
```

### State Update Rules

1. **Update immediately**: When any stage changes status
2. **Include timestamp**: ISO 8601 format for all updates
3. **Track decisions**: Log key decisions with timestamps
4. **Record current position**: Always show exact current step
5. **Note skipped stages**: Record reason for any skipped stages

## Context Loading Priority

### When Context Window is Limited

If context window is limited, prioritize loading in this order:

1. **CRITICAL**: steering-state.md (always load first)
2. **CRITICAL**: Current phase artifacts
3. **HIGH**: Previous phase outputs (summary form)
4. **MEDIUM**: Audit log (recent entries only)
5. **LOW**: Earlier phase detailed artifacts

### Context Loading Summary Message

After loading context, present summary:

```markdown
**Context Loaded:**
- State file: steering-state.md (current: DESIGN / Common Rules Design)
- DISCOVERY artifacts: purpose-analysis, domain-research, scope-definition
- DESIGN artifacts: workflow-architecture
- Pending questions: common-rules-design-questions.md (awaiting response)

**Ready to continue from**: Common Rules Design — Step 3
```

## Error Handling for Session Resumption

If artifacts are missing or corrupted during session resumption, see [error-handling.md](error-handling.md) for specific recovery procedures including:
- Missing artifacts during resumption
- Inconsistent state during resumption
- Context loading errors
- State file corruption recovery

## Multi-Session Workflow Tips

### For Long-Running Projects
- The Steering Policy Maker workflow may span multiple sessions
- Each session should start by loading the state file
- Each session should end by updating the state file
- Key decisions should be documented in both state file and audit.md

### Session Handoff
If a different AI session continues the work:
1. Load steering-state.md to understand current position
2. Load audit.md to understand decision history
3. Load artifacts from most recent phase
4. Present Welcome Back prompt to user
5. Confirm understanding before proceeding
