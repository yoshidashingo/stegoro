# Proposal Writer Process Overview

## Purpose

This file provides a high-level overview of the proposal creation workflow, explaining how phases connect, how decisions flow between stages, and how the adaptive execution model works.

---

## Workflow Architecture

The Proposal Writer follows a four-phase sequential process with adaptive stage execution:

### Phase 1: ANALYSIS
**Goal**: Fully understand the RFP before writing anything.

The Analysis phase ensures no requirement is missed and establishes the compliance baseline. All three stages are mandatory because skipping analysis leads to non-compliant proposals.

**Input**: RFP document(s) from the user
**Output**: RFP summary, categorized requirements, compliance matrix

### Phase 2: PLANNING
**Goal**: Design the response strategy before drafting content.

The Planning phase translates RFP requirements into a structured proposal plan. Win Strategy is conditional because sole-source procurements don't require competitive differentiation.

**Input**: Analysis phase artifacts
**Output**: Proposal structure, win strategy (if applicable), content plan

### Phase 3: WRITING
**Goal**: Create compelling, compliant proposal content.

The Writing phase produces actual proposal sections. Management Response and Pricing Framework are conditional because not all RFPs require them.

**Input**: Planning phase artifacts + user-provided content (capabilities, past performance, pricing data)
**Output**: Complete draft proposal sections

### Phase 4: REVIEW
**Goal**: Verify compliance and quality before submission.

The Review phase ensures the proposal meets all RFP requirements, maintains consistent quality, and is ready for submission. All stages are mandatory because unreviewed proposals carry unacceptable risk.

**Input**: All written proposal sections + compliance matrix
**Output**: Compliance check report, quality review report, final assembled proposal

---

## Adaptive Execution Model

### Stage Classification

Each stage has an execution classification:

**ALWAYS**: Stage executes in every workflow run
- RFP Intake, Requirements Extraction, Compliance Matrix
- Proposal Structure Design, Content Planning
- Executive Summary, Technical Response
- Compliance Check, Quality Review, Final Assembly

**CONDITIONAL**: Stage executes only when criteria are met
- Win Strategy — executes for competitive bids only
- Management Response — executes when RFP requests organizational/staffing information
- Pricing Framework — executes when RFP requests cost/pricing proposal

### Depth Adaptation

Stages that always execute still adapt their depth:
- **Minimal**: Simple RFP with clear, well-structured requirements
- **Standard**: Moderate complexity with multiple evaluation areas
- **Comprehensive**: Complex RFP with cross-referenced requirements, multiple volumes, strict compliance

### Decision Points

At every stage completion, the user chooses:
- **A) Request Changes** — Revise the current stage output
- **B) Continue** — Proceed to the next stage

This ensures the user maintains full control while the workflow progresses systematically.

---

## Data Flow Between Phases

```
ANALYSIS                 PLANNING                 WRITING                  REVIEW
+----------------+      +----------------+      +----------------+      +----------------+
| RFP Summary    |----->| Proposal       |----->| Executive      |----->| Compliance     |
| Requirements   |----->| Structure      |----->| Summary        |----->| Check          |
| Compliance     |----->| Win Strategy   |----->| Technical      |----->| Quality        |
| Matrix         |----->| Content Plan   |----->| Response       |----->| Review         |
+----------------+      +----------------+      | Management     |      | Final          |
                                                | Response       |      | Assembly       |
                                                | Pricing        |      +----------------+
                                                | Framework      |
                                                +----------------+
```

Each phase builds on prior artifacts. Later phases reference earlier outputs to maintain consistency and traceability.

---

## State Management

### Proposal State File (`proposal-state.md`)
Tracks overall workflow progress:
- Current phase and stage
- Completed/skipped stages with timestamps
- Key decisions made
- Conditional stage execution results

### Audit File (`audit.md`)
Logs all interactions:
- User inputs (complete, raw, never summarized)
- AI responses and actions taken
- Timestamps in ISO 8601 format
- Stage context for each entry

---

## Implementation Guidelines

1. **Always start with ANALYSIS** — Never skip to writing without understanding the RFP
2. **Respect phase ordering** — Phases execute sequentially; do not jump ahead
3. **Honor stage conditions** — Check conditional stage criteria before executing
4. **Maintain traceability** — Every proposal section should trace back to an RFP requirement
5. **Log everything** — Append to audit.md after every interaction
6. **Validate before creating** — Check content validation rules before writing any file

## Error Handling

If a phase cannot complete (e.g., RFP is incomplete, requirements are contradictory):
1. Identify the specific issue
2. Present the issue to the user with options
3. Log the issue in audit.md
4. Do not proceed until the issue is resolved

See `common/error-handling.md` for detailed error recovery procedures.

## References

- `common/content-validation.md` — Content validation rules
- `common/question-format-guide.md` — Question formatting standards
- `common/session-continuity.md` — Session resumption procedures
- `common/error-handling.md` — Error handling procedures
- `common/terminology.md` — Standardized terminology
