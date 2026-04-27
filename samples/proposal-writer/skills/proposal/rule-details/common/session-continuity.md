# Session Continuity

## Purpose

This file defines how the proposal creation workflow maintains state across sessions, enabling seamless resumption after interruptions. Session continuity ensures no work is lost and the workflow can resume from the exact point of interruption.

---

## State Tracking

### Proposal State File (`proposal-state.md`)

The state file is the single source of truth for workflow progress. It is created at workflow start and updated after every stage completion.

**State File Format**:

```markdown
# Proposal Writer — State

## Project Information
- **RFP Title**: [Title from RFP document]
- **Issuing Organization**: [Organization name]
- **Submission Deadline**: [Date]
- **RFP Type**: [IT Services / Consulting / Construction / etc.]
- **Competitive Context**: [Sole Source / Competitive Bid]
- **Created**: [ISO timestamp]
- **Last Updated**: [ISO timestamp]

## Phase Progress

### ANALYSIS PHASE
- [x/space] RFP Intake — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Requirements Extraction — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Compliance Matrix — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]

### PLANNING PHASE
- [x/space] Proposal Structure Design — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Win Strategy — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Content Planning — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]

### WRITING PHASE
- [x/space] Executive Summary — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Technical Response — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Management Response — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Pricing Framework — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]

### REVIEW PHASE
- [x/space] Compliance Check — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Quality Review — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] Final Assembly — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]

## Current Position
- **Phase**: [Current phase]
- **Stage**: [Current stage]
- **Step**: [Current step within stage]
- **Status**: [What's happening / what's awaited]

## Key Decisions
- [timestamp] [Decision description]

## Conditional Stage Decisions
- Win Strategy: [EXECUTE / SKIP] — Reason: [reason]
- Management Response: [EXECUTE / SKIP] — Reason: [reason]
- Pricing Framework: [EXECUTE / SKIP] — Reason: [reason]
```

---

## Session Resumption Procedure

### Step 1: Detect Previous Session
1. Check for existing `proposal-docs/proposal-state.md`
2. If found: Begin resumption flow
3. If not found: Start new workflow (display welcome message)

### Step 2: Load State
1. Read `proposal-state.md` completely
2. Identify current phase, stage, and step
3. Verify state file integrity (no missing fields, valid timestamps)

### Step 3: Validate Artifacts
1. Check that all artifacts for completed stages actually exist
2. For each completed stage, verify the output file is present and non-empty
3. If artifacts missing but stage marked complete: Flag inconsistency

### Step 4: Load Context
1. Load artifacts from completed stages that are relevant to the current stage
2. Load the most recent audit.md entries for context
3. Load any pending question files awaiting answers

### Step 5: Present Resumption Summary

```markdown
# Session Resumed

**RFP**: [RFP Title]
**Last Active**: [Last Updated timestamp]

## Progress Summary
- ANALYSIS: [Complete / In Progress / Pending]
- PLANNING: [Complete / In Progress / Pending]
- WRITING: [Complete / In Progress / Pending]
- REVIEW: [Complete / In Progress / Pending]

## Current Position
- **Phase**: [Phase name]
- **Stage**: [Stage name]
- **Status**: [What was happening when session ended]

## Ready to Continue

**A) Resume from [current position]** — Continue where we left off
**B) Review previous work** — Review completed stages before continuing
```

---

## State Update Rules

### When to Update State
1. **Stage start**: Mark stage as IN PROGRESS
2. **Stage completion**: Mark stage as COMPLETED with timestamp
3. **Stage skip**: Mark stage as SKIPPED with reason
4. **Key decisions**: Add to Key Decisions section
5. **Conditional evaluations**: Record Execute/Skip decision with reason

### How to Update State
1. Read current `proposal-state.md`
2. Update the relevant fields
3. Update "Last Updated" timestamp
4. Append to the file (never overwrite completely)

---

## Error Recovery During Resumption

### State File Missing
- Check if `proposal-docs/` directory exists
- If artifacts exist but no state file: Reconstruct state from artifacts
- If no artifacts: Start new workflow

### State File Corrupted
- Create backup: `proposal-state.md.backup`
- Ask user which phase they're in
- Reconstruct state from existing artifacts
- Resume from validated position

### Artifacts Missing for Completed Stage
- Mark stage as incomplete
- Ask user whether to re-execute the stage or provide information manually
- Log recovery action in audit.md

See `common/error-handling.md` for comprehensive error handling procedures.

---

## Implementation Guidelines

1. **Update state immediately**: Never defer state updates to later interactions
2. **Validate before resuming**: Always verify artifact integrity before continuing
3. **Preserve user context**: Include enough detail in state for meaningful resumption
4. **Log all resumptions**: Record session resumption in audit.md with timestamp

## References

- `common/error-handling.md` — Error recovery procedures
- `common/process-overview.md` — Workflow phase descriptions
- `common/quality-standards.md` — Quality dimensions for state validation
