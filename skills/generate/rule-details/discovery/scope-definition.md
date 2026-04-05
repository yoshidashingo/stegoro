# Scope Definition — DISCOVERY Phase

## Purpose
Define the boundaries, size, and structure of the steering policy set to be generated. This stage synthesizes all DISCOVERY findings into a concrete scope that guides the DESIGN phase.

## Prerequisites
- Purpose Analysis completed and approved
- Domain Research completed and approved
- Stakeholder Identification completed or explicitly skipped
- All DISCOVERY phase artifacts available

## Execution Classification
**Type**: ALWAYS
This stage always executes as it establishes the definitive scope for the remaining phases.

---

## Execution Steps

### Step 1: Load All DISCOVERY Artifacts
**Action**: Load and review all prior DISCOVERY phase outputs
**Input**: All DISCOVERY artifacts
**Output**: Consolidated understanding

Load and review:
1. Purpose analysis summary (agent type, capabilities, complexity)
2. Domain research summary (best practices, standards, pitfalls, terminology)
3. Stakeholder map (if created) (user types, interaction patterns, conflicts)

### Step 2: Define Policy Set Boundaries
**Action**: Explicitly define what's in scope and what's out of scope
**Input**: Consolidated DISCOVERY findings
**Output**: Scope boundary document

#### In-Scope Definition
For each of these areas, explicitly state what the policy set will cover:

1. **Workflow Coverage**: Which phases and stages will be included
2. **Quality Coverage**: Which quality mechanisms will be implemented
3. **User Coverage**: Which user types will be supported
4. **Domain Coverage**: Which domain aspects will be addressed
5. **Error Coverage**: Which error scenarios will be handled
6. **Integration Coverage**: What external systems or processes are addressed

#### Out-of-Scope Definition
Explicitly state what the policy set will NOT cover:

1. **Excluded Functionality**: What the target agent won't do
2. **Excluded Users**: What user types are not supported
3. **Excluded Domains**: What domain aspects are beyond scope
4. **Excluded Quality Levels**: What quality aspirations are deferred
5. **Future Considerations**: What could be added later

### Step 3: Estimate Phase and Stage Structure
**Action**: Estimate the number of phases and stages for the target agent
**Input**: Agent type classification + complexity assessment
**Output**: Preliminary structure estimate

#### Estimation Guidelines by Agent Type

**Process Agent**:
- Typical: 3-5 phases with 3-7 stages each
- Total stages: 12-25
- Common phases: Planning, Execution, Verification, (Delivery)
- Rule files: 25-35

**Task Agent**:
- Typical: 2-3 phases with 2-4 stages each
- Total stages: 5-10
- Common phases: Setup, Execution, Reporting
- Rule files: 15-22

**Analytical Agent**:
- Typical: 2-4 phases with 2-5 stages each
- Total stages: 6-15
- Common phases: Input Analysis, Processing, Output Formatting, (Validation)
- Rule files: 18-28

**Hybrid Agent**:
- Typical: 3-6 phases with variable stages
- Total stages: 10-30
- Common phases: Context Assessment, [Mode-specific], Synthesis
- Rule files: 22-40

### Step 4: Estimate Rule File Inventory
**Action**: Create a preliminary list of expected rule files
**Input**: Phase/stage structure + common rules requirements
**Output**: File inventory estimate

#### Standard Common Rules (Always Included)
1. `welcome-message.md`
2. `process-overview.md`
3. `question-format-guide.md`
4. `content-validation.md`
5. `session-continuity.md`
6. `error-handling.md`
7. `overconfidence-prevention.md`
8. `terminology.md`
9. `quality-standards.md`
10. `output-structure-patterns.md`

#### Domain-Specific Common Rules (As Needed)
- Identified from domain research findings
- Typically 0-5 additional common rules

#### Phase-Specific Rules
- One rule file per stage (minimum)
- Complex stages may have additional supporting files

### Step 5: Define Quality Level Target
**Action**: Set the quality standard for the generated policy set
**Input**: Domain risk profile + complexity assessment
**Output**: Quality target definition

#### Quality Levels

| Aspect | Basic | Standard | Premium |
|--------|-------|----------|---------|
| **11 Dimensions** | 8+ PASS | 10+ PASS | All 11 PASS |
| **File Depth** | ~100 lines avg | ~150 lines avg | ~200+ lines avg |
| **Error Handling** | Major scenarios | Most scenarios | All scenarios |
| **Domain Specificity** | Generic + adapted | Moderately specific | Highly specific |
| **Cross-References** | Basic | Comprehensive | Exhaustive |
| **Examples** | Minimal | Moderate | Extensive |

Select quality level based on:
- Domain risk (high risk → Premium)
- Complexity (complex → Standard or Premium)
- User expectations (stated quality needs)
- Agent criticality (important agent → higher quality)

### Step 6: Create Preliminary Directory Structure
**Action**: Design the directory layout for the generated policy set
**Input**: Phase/stage estimates + file inventory
**Output**: Directory structure document

#### Directory Structure Template

```
.<agent-name>/
├── <agent-name>-rules/
│   └── core-workflow.md
└── <agent-name>-rule-details/
    ├── common/
    │   ├── welcome-message.md
    │   ├── process-overview.md
    │   ├── question-format-guide.md
    │   ├── content-validation.md
    │   ├── session-continuity.md
    │   ├── error-handling.md
    │   ├── overconfidence-prevention.md
    │   ├── terminology.md
    │   ├── quality-standards.md
    │   ├── output-structure-patterns.md
    │   └── [domain-specific-common-rules...]
    ├── <phase-1-name>/
    │   ├── <stage-1>.md
    │   ├── <stage-2>.md
    │   └── ...
    ├── <phase-2-name>/
    │   ├── <stage-1>.md
    │   └── ...
    └── <phase-N-name>/
        └── ...
```

### Step 7: Compile Scope Summary
**Action**: Compile all scope decisions into a comprehensive summary
**Input**: All scope definition outputs
**Output**: Scope summary document

### Step 8: Generate Scope Questions (If Needed)
**Action**: Create questions about scope decisions that need user input
**Input**: Scope summary with uncertainties
**Output**: Scope question file

Create `steering-docs/discovery/scope-definition-questions.md` if:
- Boundary decisions could reasonably go either way
- File count or structure has multiple valid options
- Quality level needs user preference input
- Trade-offs between scope and effort need user decision

### Step 9: Present Scope Definition
**Action**: Present complete scope definition for approval
**Input**: All scope outputs
**Output**: Formatted scope presentation for user review

---

## Output Artifacts

### Scope Definition Document
- **File**: `steering-docs/discovery/scope-definition.md`
- **Content**: Complete scope with boundaries, estimates, and structure
- **Format**:

```markdown
# Scope Definition

## Target Agent Summary
- **Name**: [agent name]
- **Type**: [Process / Task / Analytical / Hybrid]
- **Domain**: [domain]
- **Complexity**: [Simple / Standard / Complex]

## Scope Boundaries

### In Scope
| Area | Coverage | Notes |
|------|----------|-------|
| Workflow | [description] | |
| Quality | [description] | |
| Users | [description] | |
| Domain | [description] | |
| Errors | [description] | |

### Out of Scope
- [Exclusion 1]: [Reason]
- [Exclusion 2]: [Reason]
- [Future Consideration 1]: [When to add]

## Estimated Structure
- **Phases**: [N] (including PACKAGING phase for Complex agents)
- **Total Stages**: [N] (ALWAYS: [N], CONDITIONAL: [N])
- **Common Rule Files**: [N] (standard: 10, domain-specific: [N], + implementation-knowhow)
- **Phase Rule Files**: [N] (including packaging/ for PACKAGING phase)
- **Plugin Files**: [N] (agents, skills, commands, plugin config — if PACKAGING included)
- **Total Files**: [N]
- **Estimated Total Lines**: [N]

### Baseline Measurement (Brush-Up Only)
For existing policy improvements, measure actual file sizes before estimating:
```bash
wc -l <existing-directory>/**/*.md
```
Use measured values (not estimates) for current line counts. Calculate improvement delta per file.

## Preliminary Phase Structure
### Phase 1: [Name]
- Stage 1: [Name] (ALWAYS/CONDITIONAL)
- Stage 2: [Name] (ALWAYS/CONDITIONAL)

### Phase 2: [Name]
- Stage 1: [Name] (ALWAYS/CONDITIONAL)

[Continue for all phases]

## Quality Target
- **Level**: [Basic / Standard / Premium]
- **15 Dimensions Target**: [N/15 PASS] (11 AI-DLC standard + 4 domain-specific)
- **Key Quality Focus Areas**: [List]

## Directory Structure
[Directory tree as designed in Step 6]

## Effort Estimate
- **DESIGN Phase**: [Approximate effort description]
- **GENERATION Phase**: [Approximate effort description]
- **REFINEMENT Phase**: [Approximate effort description]
```

---

## Completion Message

### REVIEW REQUIRED

**Scope Definition is complete.** Here's a summary:

- **Target Agent**: [name] ([type])
- **Estimated Structure**: [N] phases, [N] stages
- **Estimated Files**: [N] files (~[N] total lines)
- **Quality Target**: [level]

Key scope decisions:
- [Decision 1]
- [Decision 2]
- [Decision 3]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the scope definition based on your feedback
**B) Continue to DESIGN Phase** — Proceed to design the workflow architecture

---

### PHASE COMPLETE: DISCOVERY

**All stages in the DISCOVERY phase are complete.**

**Summary of outputs:**
- Purpose Analysis: [agent type] agent classified
- Domain Research: [N] best practices, [N] pitfalls identified
- Stakeholder Identification: [Completed/Skipped] ([reason])
- Scope Definition: [N] phases, [N] stages, [N] files estimated

### WHAT'S NEXT

The next phase is **DESIGN**: Architect the steering policy structure.

**A) Proceed to DESIGN Phase** — Continue the workflow
**B) Review DISCOVERY outputs** — Review before moving forward

---

## Error Handling

### Scope Too Large
- **Issue**: Estimated file count or complexity exceeds practical limits
- **Solution**: Propose scope reduction options (phase the work, reduce quality target)
- **Workaround**: Generate core workflow + common rules first, add phase rules incrementally

### Scope Too Small
- **Issue**: Estimated scope seems insufficient for the agent's needs
- **Solution**: Review with user whether more phases or stages are needed
- **Workaround**: Generate with current scope, expand during REFINEMENT if gaps found

### Uncertain Boundaries
- **Issue**: Can't determine clear in-scope/out-of-scope boundaries
- **Solution**: Create scope questions for user decision
- **Do Not Proceed**: Until boundaries are defined

### Conflicting Scope Requirements
- **Issue**: Different stakeholders want different scope
- **Solution**: Present trade-offs, ask user to prioritize
- **Fallback**: Default to broader scope, note that refinement will tighten boundaries

### Scope Changes After Approval
- **Issue**: User wants to change scope after DESIGN has started
- **Solution**: Assess impact on existing design work, propose change plan
- **Recovery**: Restart from Scope Definition if change is fundamental

## References
- `discovery/purpose-analysis.md` — Agent classification
- `discovery/domain-research.md` — Domain findings
- `discovery/stakeholder-identification.md` — Stakeholder needs
- `common/quality-standards.md` — Quality level definitions
- `common/output-structure-patterns.md` — File structure patterns
- `common/terminology.md` — Term definitions
