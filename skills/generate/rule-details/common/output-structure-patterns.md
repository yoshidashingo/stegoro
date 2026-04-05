# Output Structure Patterns

## Purpose

This file defines the standardized structural patterns that ALL generated steering policy files must follow. These patterns ensure consistency across the policy set and make files predictable for both the target agent and human reviewers.

---

## Core Workflow File Pattern

The core workflow file (`core-workflow.md`) is the master orchestrator. It follows this structure:

```markdown
# PRIORITY: This workflow OVERRIDES all other built-in workflows
# [Context-specific override statement]

## Adaptive Workflow Principle
[Adaptation criteria specific to target agent]

## MANDATORY: Rule Details Loading
[List of common rules to load at startup]

## MANDATORY: Content Validation
[Reference to content-validation.md]

## MANDATORY: Question File Format
[Reference to question-format-guide.md]

## MANDATORY: Custom Welcome Message
[Reference to welcome-message.md]

# [Agent Name] — Adaptive Workflow

---

# [PHASE 1 NAME] PHASE

**Purpose**: [What this phase achieves]
**Focus**: [Key focus area]
**Stages in [PHASE 1 NAME] PHASE**:
- [Stage 1] (ALWAYS/CONDITIONAL)
- [Stage 2] (ALWAYS/CONDITIONAL)
- ...

---

## [Stage 1 Name] (ALWAYS/CONDITIONAL EXECUTE)

[For CONDITIONAL stages, include:]
**Execute IF**:
- [Condition 1]
- [Condition 2]

**Skip IF**:
- [Condition 1]
- [Condition 2]

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `[phase]/[stage].md`
3. [Stage-specific execution steps]
4. **Wait for Explicit Approval**: [Approval prompt] — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

---

[Repeat for each phase and stage]

---

## Key Principles
[List of principles specific to the target agent]

## MANDATORY: Plan-Level Checkbox Enforcement
[Checkbox tracking rules]

## Prompts Logging Requirements
[Audit logging rules with format]

## Directory Structure
[Generated output directory structure]
```

---

## Common Rule File Pattern

All common rule files follow this structure:

```markdown
# [Rule Title]

## Purpose
[One paragraph explaining what this rule governs and why it matters]

---

## [Main Content Sections]
[Organized by topic with clear headings]

### [Subsection]
[Content with examples where appropriate]

---

## Implementation Guidelines
[How to apply this rule in practice]

## Error Handling
[What to do when this rule is violated or cannot be followed]

## References
[Links to related common rules or external standards]
```

---

## Phase Rule File Pattern

All phase-specific rule files follow this structure:

```markdown
# [Stage Name] — [Phase Name] Phase

## Purpose
[What this stage achieves in the context of the workflow]

## Prerequisites
- [What must be completed before this stage]
- [Artifacts that must be loaded]
- [Common rules to reference]

## Execution Classification
**Type**: ALWAYS / CONDITIONAL
[For CONDITIONAL: Execute IF / Skip IF criteria]

## Adaptive Depth
[If applicable: Minimal / Standard / Comprehensive descriptions]

---

## Execution Steps

### Step 1: [Step Title]
**Action**: [What to do]
**Input**: [What data/context to use]
**Output**: [What this step produces]
**Validation**: [How to verify correctness]

### Step 2: [Step Title]
[Continue for all steps]

---

## Question Generation
[When and what questions to ask during this stage]

### Question Categories
- [Category 1]: [Types of questions]
- [Category 2]: [Types of questions]

### Question File Template
[Template for generating stage-specific questions]

---

## Output Artifacts
[List of files/artifacts this stage produces]

### [Artifact 1 Name]
- **File**: `[path/filename.md]`
- **Content**: [Description of what the artifact contains]
- **Format**: [Structure of the artifact]

---

## Completion Message

### REVIEW REQUIRED

**[Stage Name] is complete.** Here's what was produced:

[Summary of outputs]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the [output] based on your feedback
**B) Continue to [Next Stage]** — Proceed to the next stage

---

## Error Handling
[Stage-specific error scenarios and recovery procedures]
[Reference to common/error-handling.md for general procedures]
```

---

## Question File Pattern

```markdown
# [Phase/Stage Name] Questions

Please answer the following questions to help [context-specific purpose].

## Question 1
[Clear, specific question text]

A) [First meaningful option]
B) [Second meaningful option]
C) [Third meaningful option — if applicable]
D) [Fourth meaningful option — if applicable]
E) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 2
[Continue for all questions]
```

---

## Clarification Question File Pattern

```markdown
# [Phase/Stage Name] Clarification Questions

I detected [contradictions/ambiguities] in your responses that need clarification:

## [Issue Type] 1: [Brief Description]
You indicated "[Answer A]" (Q[X]:[Letter]) but also "[Answer B]" (Q[Y]:[Letter]).
[Explanation of why these are contradictory/ambiguous]

### Clarification Question 1
[Specific question to resolve the issue]

A) [Option resolving toward first answer]
B) [Option resolving toward second answer]
C) [Option providing middle ground]
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Plan File Pattern

```markdown
# [Stage Name] — Generation Plan

## Context
[Summary of relevant prior artifacts and decisions]

## Plan Steps

- [ ] Step 1: [Step description]
  - Input: [What's needed]
  - Output: [What will be produced]
  - Validation: [How to verify]

- [ ] Step 2: [Step description]
  - Input: [What's needed]
  - Output: [What will be produced]
  - Validation: [How to verify]

[Continue for all steps]

## Approval Required
Please review this plan and confirm:
**A) Approve plan** — Proceed with generation
**B) Request changes** — Modify the plan before proceeding
```

---

## Validation Report Pattern

```markdown
# [Validation Type] Report

## Summary
- **Total Items Checked**: [N]
- **Passed**: [N]
- **Failed**: [N]
- **Warnings**: [N]

## Detailed Results

### [Category 1]
| Item | Status | Details |
|------|--------|---------|
| [Item 1] | PASS/FAIL | [Details] |
| [Item 2] | PASS/FAIL | [Details] |

### [Category 2]
[Continue for all categories]

## Issues Found
[Detailed description of each failure]

## Recommendations
[Actions to resolve issues]

## Overall Status: [PASS / FAIL]
```

---

## State File Pattern

```markdown
# [Agent Name] — Steering Policy Maker State

## Project Information
- **Target Agent**: [agent-name]
- **Agent Type**: [Process/Task/Analytical/Hybrid]
- **Domain**: [domain description]
- **Created**: [ISO timestamp]
- **Last Updated**: [ISO timestamp]

## Phase Progress

### [PHASE 1] PHASE
- [x/space] [Stage 1] — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]
- [x/space] [Stage 2] — [COMPLETED/IN PROGRESS/PENDING/SKIPPED] [timestamp]

[Continue for all phases]

## Current Position
- **Phase**: [Current phase]
- **Stage**: [Current stage]
- **Step**: [Current step within stage]
- **Status**: [What's happening / what's awaited]

## Key Decisions
- [timestamp] [Decision description]
```

---

## Audit File Pattern

```markdown
# [Agent Name] — Steering Policy Creation Audit Log

## [Stage/Interaction Type]
**Timestamp**: [ISO 8601 timestamp]
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[AI's response or action taken]"
**Context**: [Stage, action, or decision made]

---

[Continue for each interaction]
```

---

## Completion Message Patterns

### Stage Completion (Standard 2-Option)

```markdown
---

### REVIEW REQUIRED

**[Stage Name] is complete.** Here's a summary of what was produced:

- [Output 1 description]
- [Output 2 description]
- [Output N description]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise based on your feedback
**B) Continue to [Next Stage Name]** — Proceed to the next stage

---
```

### Phase Completion

```markdown
---

### PHASE COMPLETE: [PHASE NAME]

**All stages in the [Phase Name] phase are complete.**

**Summary of outputs:**
- [Key output 1]
- [Key output 2]
- [Key output N]

### WHAT'S NEXT

The next phase is **[Next Phase Name]**: [Brief description of what the next phase does].

**A) Proceed to [Next Phase Name]** — Continue the workflow
**B) Review [Current Phase Name] outputs** — Review before moving forward

---
```

### Workflow Completion

```markdown
---

### STEERING POLICY SET COMPLETE

**The steering policy set for [Agent Name] has been generated and validated.**

**Final Deliverables:**
- Directory: `.[agent-name]/`
- Total files: [N]
- Total lines: [N]
- Quality calibration: [X/15 dimensions PASS]

**Contents:**
- Core workflow: `[agent-name]-rules/core-workflow.md`
- Common rules: [N] files in `[agent-name]-rule-details/common/`
- Phase rules: [N] files across [N] phase directories

### NEXT STEPS

1. Review the generated policy files in `.[agent-name]/`
2. Test the workflow by running the target agent with a sample task
3. Iterate on any rules that need adjustment

---
```

---

## Content Depth Guidance

Generated file depth should match the target agent's complexity:

| Depth | Target | Section Structure | Examples | Templates |
|-------|--------|-------------------|----------|-----------|
| **Light** | Simple / Task Agent | Purpose, Steps (overview), Completion Message | 0-1 | Optional |
| **Medium** | Standard Agent | Purpose, Prerequisites, Steps (detailed), Examples, Completion Message, Error Handling | 1-2 | Recommended |
| **Heavy** | Complex / Process Agent | Purpose, Prerequisites, Steps (detailed + substeps), Examples (GOOD/BAD), Completion Message, Error Handling, References | 2+ | Required |

### Line Count Guidelines by File Type

| File Type | Light | Medium | Heavy |
|-----------|-------|--------|-------|
| SKILL.md | 75-90 | 90-120 | 120-150 |
| core-workflow.md | 200-300 | 300-450 | 450-600 |
| Phase Rule file | 100-150 | 150-250 | 250-300 |
| Common Rule file | 50-100 | 100-150 | 150-200 |

---

## Plugin File Patterns

### SKILL.md Pattern (75-150 lines)

```markdown
---
name: <skill-name>
description: <one-line description>
---

## Activation Trigger
[When this skill is invoked]

## Core Rules
1. [Rule 1]
2. [Rule 2]
...

## Workflow Summary
[Phase/stage overview with references to rule-details/]

## Quality Gate
[Minimum quality criteria]

## References
- `rule-details/core-workflow.md` — Full workflow
- `rule-details/common/` — Common rules
```

### plugin.json Pattern

```json
{
  "name": "<agent-name>",
  "version": "1.0.0",
  "description": "...",
  "agents": ["agents/*.md"],
  "skills": ["skills/*/SKILL.md"],
  "commands": ["commands/*.md"],
  "rules": ["rules/*.md"]
}
```

### Agent Definition Pattern

```markdown
---
name: <agent-name>
description: <one-line description>
---

## Role
[Agent's responsibility]

## Capabilities
[What this agent can do]

## Workflow Integration
[Which phases/stages this agent handles]
```

### Command Definition Pattern

```markdown
---
name: <prefix>:<command-name>
description: <one-line description>
---

## Execution
1. [Step 1]
2. [Step 2]
```
