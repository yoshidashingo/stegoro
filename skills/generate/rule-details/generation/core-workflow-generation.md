# Core Workflow Generation — GENERATION Phase

## Purpose
Generate the target agent's master orchestrator file (`core-workflow.md`). This is the most critical file in the policy set — it defines the complete workflow, references all other files, and establishes the execution rules that govern the target agent's behavior.

## Prerequisites
- All DISCOVERY phase artifacts available and approved
- All DESIGN phase artifacts available and approved:
  - Workflow Architecture (phases, stages, dependencies)
  - Common Rules Design (rules inventory and adaptations)
  - Phase Rules Design (stage file specifications)
  - Quality Mechanisms Design (checkpoint, audit, validation systems)

## Execution Classification
**Type**: ALWAYS
This stage always executes as the core workflow file is mandatory for every policy set.

---

## Two-Part Execution

This stage uses the Planning → Generation pattern for safety and quality.

### Part 1 — Planning

#### Step 1: Load All Design Artifacts
**Action**: Load and review all DESIGN phase outputs
**Input**: All DESIGN artifacts
**Output**: Complete generation context

Verify all required information is available:
- [ ] Workflow architecture with all phases and stages defined
- [ ] Common rules inventory with all 10+ rules specified
- [ ] Phase rules inventory with all stage files specified
- [ ] Quality mechanisms with all 15 dimensions designed
- [ ] Content validation rules loaded

#### Step 2: Create Generation Plan
**Action**: Create a detailed plan for the core workflow file
**Input**: All design artifacts
**Output**: Core workflow generation plan with checkboxes

**Plan Template**:

```markdown
# Core Workflow Generation Plan

## Section 1: Header and Overrides
- [ ] Write priority override statement
- [ ] Write adaptive workflow principle
- [ ] Define MANDATORY rule loading section
- [ ] Define MANDATORY content validation section
- [ ] Define MANDATORY question format section
- [ ] Define MANDATORY welcome message section

## Section 2: Phase Definitions
[For each phase from workflow architecture:]
- [ ] Write [Phase N] header with purpose and focus
- [ ] List all stages with ALWAYS/CONDITIONAL classification
- [ ] Write separator

## Section 3: Stage Definitions
[For each stage from workflow architecture:]
- [ ] Write [Stage Name] section header with classification
- [ ] [For CONDITIONAL:] Write Execute IF criteria
- [ ] [For CONDITIONAL:] Write Skip IF criteria
- [ ] Write execution steps (numbered list)
- [ ] Include MANDATORY audit logging step
- [ ] Include rule file loading step
- [ ] Include approval gate step
- [ ] Include MANDATORY response logging step

## Section 4: Key Principles
- [ ] Write key principles list
- [ ] Write plan-level checkbox enforcement rules
- [ ] Write prompts logging requirements with format
- [ ] Write audit log format
- [ ] Write correct tool usage guidance

## Section 5: Domain Adaptation
- [ ] Write agent type classification section (if applicable)
- [ ] Write directory structure section
- [ ] Write critical rules section

## Validation
- [ ] All stages from workflow architecture included
- [ ] All file references point to files in the design inventory
- [ ] All CONDITIONAL stages have explicit criteria
- [ ] All stages have approval gates
- [ ] Completion message format is consistent
```

## Adaptive Depth — Output Templates

The generated core-workflow.md varies by target agent complexity:

| Depth | Target | Phases | Stages | Lines | Characteristics |
|-------|--------|--------|--------|-------|-----------------|
| **Light** | Simple/Task Agent | 2-3 | 6-10 | 200-300 | Stage overviews, basic approval gates |
| **Medium** | Standard Agent | 3-4 | 10-14 | 300-450 | Execution summaries, full approval gates, error handling refs |
| **Heavy** | Complex/Process Agent | 4-5 | 14-18 | 450-600 | Detailed steps, adaptive depth, error handling, repair judgment tree |

### PACKAGING Phase Generation Guide

For Complex agents, include Phase 5: PACKAGING in the generated core-workflow:
- P1: Plugin Structure Generation (ALWAYS) — plugin.json, agents, skills, commands
- P2: Automated Validation (ALWAYS) — 3-layer testing for 4 agent types
- P3: Migration Execution (CONDITIONAL) — legacy structure migration

For Simple/Task agents, PACKAGING may be omitted if user prefers policy-only output.

#### Step 3: Present Plan for Approval
**Action**: Present the generation plan to user
**Input**: Generation plan
**Output**: User approval

### Part 2 — Generation

#### Step 4: Generate Header Sections
**Action**: Write the core workflow file's header sections
**Input**: Design artifacts + approved plan
**Output**: Header sections of core-workflow.md

Generate:
1. Priority override statement (adapted from SPM's core-workflow.md pattern)
2. Adaptive workflow principle (adapted to target agent's domain)
3. MANDATORY rule details loading (list all common rule files)
4. MANDATORY content validation (reference content-validation.md)
5. MANDATORY question file format (reference question-format-guide.md)
6. MANDATORY welcome message (reference welcome-message.md)

#### Step 5: Generate Phase Sections
**Action**: Write each phase's section in the core workflow
**Input**: Workflow architecture
**Output**: Phase sections of core-workflow.md

For each phase:
1. Phase header with name
2. Purpose statement
3. Focus statement
4. Stage list with ALWAYS/CONDITIONAL classifications
5. Horizontal separator

#### Step 6: Generate Stage Sections
**Action**: Write each stage's detailed section
**Input**: Workflow architecture + phase rules design
**Output**: Stage sections of core-workflow.md

For each stage, generate:
1. Stage header with execution classification
2. For CONDITIONAL: Execute IF and Skip IF criteria
3. For two-part stages: Part 1/Part 2 description
4. Numbered execution steps including:
   - MANDATORY audit logging
   - Rule file loading instruction
   - Stage-specific execution steps
   - Content validation (where applicable)
   - Wait for Explicit Approval instruction
   - MANDATORY response logging

#### Step 7: Generate Principles and Rules Sections
**Action**: Write the key principles and operational rules
**Input**: Quality mechanisms design
**Output**: Principles sections of core-workflow.md

Generate:
1. Key Principles list (adapted from quality mechanisms design)
2. Plan-Level Checkbox Enforcement rules
3. Prompts Logging Requirements with format template
4. Audit Log Format template
5. Correct tool usage guidance

#### Step 8: Generate Domain-Specific Sections
**Action**: Write domain-specific sections
**Input**: Purpose analysis + scope definition
**Output**: Domain sections of core-workflow.md

Generate (as applicable):
1. Agent type classification / mode detection (for Hybrid agents)
2. Directory structure diagram
3. Critical rules specific to the domain
4. Any domain-specific workflow variations

#### Step 9: Validate Generated Content
**Action**: Validate the complete core workflow file
**Input**: Generated core-workflow.md
**Output**: Validation results

**MANDATORY** validation checks:
- [ ] All stages from workflow architecture are included
- [ ] All file references use correct paths
- [ ] All CONDITIONAL stages have explicit criteria
- [ ] All stages have MANDATORY audit logging steps
- [ ] All stages have approval gates with 2-option format
- [ ] Mermaid diagrams (if any) pass syntax validation
- [ ] ASCII diagrams (if any) follow standards
- [ ] Markdown syntax is correct
- [ ] No orphaned references
- [ ] Completion message format is consistent across all stages

#### Step 10: Write Core Workflow File
**Action**: Write the validated core-workflow.md to disk
**Input**: Validated content
**Output**: `<agent-name>/<agent-name>-rules/core-workflow.md`

**CRITICAL**: Create the target directory structure first if it doesn't exist.

---

## Output Artifacts

### Core Workflow File
- **File**: `<agent-name>/<agent-name>-rules/core-workflow.md`
- **Content**: Complete master orchestrator for the target agent
- **Expected Size**: 200-500 lines depending on complexity

### Generation Plan
- **File**: `steering-docs/<agent-name>/generation/core-workflow-generation-plan.md`
- **Content**: Detailed plan with checkboxes tracking progress

---

## Completion Message

### REVIEW REQUIRED

**Core Workflow Generation is complete.** Here's what was produced:

- **File**: `<agent-name>/<agent-name>-rules/core-workflow.md`
- **Line Count**: [N] lines
- **Phases Documented**: [N]
- **Stages Documented**: [N] (ALWAYS: [N], CONDITIONAL: [N])
- **Approval Gates**: [N]
- **File References**: [N] (all validated)

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the core workflow based on your feedback
**B) Continue to Common Rules Generation** — Proceed to generate common rule files

---

## Error Handling

### Design Artifact Missing
- **Issue**: A required design artifact is missing or incomplete
- **Solution**: Return to DESIGN phase to complete the missing artifact
- **Do Not Proceed**: Until all design artifacts are available

### File Reference Mismatch
- **Issue**: Core workflow references files that aren't in the design inventory
- **Solution**: Either add the file to the inventory or fix the reference
- **Validation**: Re-validate all references after fix

### Content Exceeds Practical Length
- **Issue**: Generated core workflow is too long (500+ lines)
- **Solution**: Move detailed content to referenced rule files, keep core workflow concise
- **Recovery**: Restructure to use more references and less inline content

### Inconsistent Classification
- **Issue**: Stage classification in core workflow doesn't match design
- **Solution**: Align with the approved workflow architecture design
- **Recovery**: Fix inconsistencies and re-validate

## References
- `design/workflow-architecture.md` — Phase/stage structure
- `design/common-rules-design.md` — Common rules inventory
- `design/phase-rules-design.md` — Stage file specifications
- `design/quality-mechanisms-design.md` — Quality mechanisms
- `common/content-validation.md` — Validation rules
- `common/output-structure-patterns.md` — Core workflow file pattern
