# Phase Rules Design — DESIGN Phase

## Purpose
Design the detailed content specifications for each phase-specific rule file. This stage creates the blueprint for every stage rule file that will be generated, ensuring comprehensive coverage of the target agent's workflow.

## Prerequisites
- All DISCOVERY phase artifacts available and approved
- Workflow Architecture completed and approved
- Common Rules Design completed and approved

## Execution Classification
**Type**: ALWAYS
This stage always executes as phase-specific rules are the core content of any steering policy set.

---

## Execution Steps

### Step 1: Load Prior Context
**Action**: Load all relevant prior artifacts
**Input**: All DISCOVERY + prior DESIGN artifacts
**Output**: Complete design context

Key context:
- Workflow architecture (phases, stages, dependencies, checkpoints)
- Common rules design (what cross-phase rules will be available)
- Domain best practices and pitfalls
- Stakeholder interaction patterns
- Quality target and scope

### Step 2: Create Phase Rules Inventory
**Action**: List all phase rule files that need to be created
**Input**: Workflow architecture stage inventory
**Output**: Complete file list

For each phase, list:
1. All stage rule files (one per stage minimum)
2. Any additional supporting files per stage
3. Expected approximate line count per file

### Step 3: Design Individual Stage Rule Files
**Action**: Create detailed design specifications for each rule file
**Input**: Stage definitions + domain context
**Output**: Rule file design specifications

For EACH stage rule file, design the following:

#### A. File Header Design
- **Title**: Stage name + phase context
- **Purpose Statement**: What the stage achieves (1-2 sentences)
- **Prerequisites**: What must be completed/loaded before this stage
- **Execution Classification**: ALWAYS or CONDITIONAL with criteria

#### B. Execution Steps Design
For each step within the stage:
1. **Step Name**: Clear, action-oriented
2. **Action Description**: What to do
3. **Input Specification**: What data/context is needed
4. **Output Specification**: What this step produces
5. **Validation Criteria**: How to verify correctness
6. **Domain-Specific Content**: Any domain-specific actions or checks

#### C. Question Generation Design
- **Question Categories**: What types of questions this stage might ask
- **Typical Questions**: Example questions specific to this stage
- **Contradiction Patterns**: What contradictions to look for in responses
- **Follow-up Triggers**: When to ask follow-up questions

#### D. Output Artifact Design
- **Artifact Files**: What files this stage produces
- **Artifact Format**: Structure and format of each artifact
- **Quality Criteria**: How to verify artifact quality

#### E. Completion Message Design
- **Summary Content**: What to summarize for the user
- **2-Option Format**: "Request Changes" or "Continue to [Next Stage]"
- **Next Stage Reference**: Which stage follows

#### F. Error Handling Design
- **Stage-Specific Errors**: What can go wrong in this stage
- **Recovery Procedures**: How to recover from each error
- **Escalation Points**: When to ask the user for help

### Step 4: Design Inter-Stage References
**Action**: Map how stage rule files reference each other
**Input**: Stage designs + dependency map
**Output**: Cross-reference matrix

For each stage rule file:
- Which common rules does it reference?
- Which previous stage outputs does it need?
- Which subsequent stages depend on its output?
- What cross-references appear in its content?

### Step 5: Design Conditional Logic
**Action**: Design the conditional execution logic for CONDITIONAL stages
**Input**: Stage classifications + domain context
**Output**: Conditional logic specifications

For each CONDITIONAL stage:
1. **Execute IF conditions**: Specific, testable conditions
2. **Skip IF conditions**: Explicit skip criteria
3. **Assessment Process**: How to evaluate conditions
4. **Fallback**: What happens if conditions are ambiguous

### Step 6: Design Two-Part Stages (If Applicable)
**Action**: Design stages that have Planning + Generation parts
**Input**: Stage complexity assessment
**Output**: Two-part stage specifications

Some stages may benefit from the AI-DLC "two-part" pattern:
- **Part 1 — Planning**: Create a plan with checkboxes, get user approval
- **Part 2 — Generation**: Execute the approved plan

Consider two-part design when:
- Stage produces significant, complex artifacts
- User review before generation adds value
- The generation step is not easily reversible
- Stage has multiple possible approaches

### Step 7: Verify Coverage Completeness
**Action**: Verify all workflow paths are covered by rule files
**Input**: Workflow architecture + rule file inventory
**Output**: Coverage verification report

Check:
- Every stage in the workflow has a corresponding rule file designed
- Every CONDITIONAL path has both execute and skip scenarios handled
- Every phase transition is documented
- Every approval gate has corresponding completion message
- No orphaned stages (stages not reachable from any path)
- No dead-end stages (stages with no next step defined)

### Step 8: Generate Phase Rules Design Questions (If Needed)
**Action**: Create questions about design decisions
**Input**: Design specifications with decision points
**Output**: Phase rules design question file

### Step 9: Present Phase Rules Design
**Action**: Present complete phase rules design for approval
**Input**: All design specifications
**Output**: Formatted design presentation

---

## Output Artifacts

### Phase Rules Design Document
- **File**: `steering-docs/<agent-name>/design/phase-rules-design.md`
- **Content**: Complete design specifications for all phase rule files
- **Format**:

```markdown
# Phase Rules Design

## File Inventory
| Phase | Stage | File Name | Classification | Est. Lines |
|-------|-------|-----------|---------------|------------|
| [Phase 1] | [Stage 1] | [filename].md | ALWAYS | ~[N] |
| [Phase 1] | [Stage 2] | [filename].md | CONDITIONAL | ~[N] |
| [Phase 2] | [Stage 1] | [filename].md | ALWAYS | ~[N] |

**Total Phase Rule Files**: [N]
**Estimated Total Lines**: [N]

## Phase 1: [Name]

### Stage: [Stage Name]
**File**: `[phase-dir]/[filename].md`
**Classification**: ALWAYS/CONDITIONAL
**Purpose**: [What this stage achieves]

#### Execution Steps (Outline)
1. [Step 1 name]: [Brief description]
2. [Step 2 name]: [Brief description]
3. [Step N name]: [Brief description]

#### Questions to Ask
- [Category 1]: [Types of questions]
- [Category 2]: [Types of questions]

#### Output Artifacts
- [Artifact 1]: [Description and format]

#### Error Scenarios
- [Error 1]: [Brief description and recovery]

#### Completion Message
- Summary: [What to present]
- Next: [Next stage name]

[Continue for each stage in each phase]

## Cross-Reference Matrix
| Source File | References |
|-------------|-----------|
| [Stage 1] | common/question-format-guide.md, [Stage 0 output] |
| [Stage 2] | common/terminology.md, [Stage 1 output] |

## Coverage Verification
- [ ] All stages have corresponding rule file designs
- [ ] All CONDITIONAL stages have execute/skip criteria
- [ ] All phase transitions documented
- [ ] All approval gates have completion messages
- [ ] No orphaned or dead-end stages
- [ ] All cross-references valid
```

---

## Completion Message

### REVIEW REQUIRED

**Phase Rules Design is complete.** Here's a summary:

- **Total Phase Rule Files Designed**: [N]
- **Phase 1 ([name])**: [N] stage files
- **Phase 2 ([name])**: [N] stage files
- **Phase N ([name])**: [N] stage files
- **Two-Part Stages**: [N]
- **Conditional Stages**: [N]
- **Coverage**: [All paths covered / Issues found]

Key design decisions:
- [Decision 1]
- [Decision 2]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the phase rules design based on your feedback
**B) Continue to Quality Mechanisms Design** — Proceed to design quality assurance mechanisms

---

## Error Handling

### Coverage Gaps Found
- **Issue**: Some workflow paths don't have corresponding rule files
- **Solution**: Design additional rule files to cover gaps
- **Do Not Proceed**: Until all paths are covered

### Stage Too Complex for Single File
- **Issue**: A stage requires more content than fits in one manageable file
- **Solution**: Split into sub-stages or use the two-part pattern
- **Recovery**: Create supporting files that the main stage file references

### Design Too Generic
- **Issue**: Stage designs lack domain-specific content
- **Solution**: Review domain research and add specific domain elements
- **Do Not Proceed**: Until designs are sufficiently domain-specific

### Inconsistent Detail Levels
- **Issue**: Some stages have detailed designs while others are thin
- **Solution**: Equalize detail across all stage designs, especially for ALWAYS stages
- **Recovery**: Expand thin designs using domain research as input

## References
- `design/workflow-architecture.md` — Phase/stage structure and dependencies
- `design/common-rules-design.md` — Available common rules
- `discovery/domain-research.md` — Domain practices and pitfalls
- `discovery/scope-definition.md` — Quality target and estimates
- `common/output-structure-patterns.md` — Standard file patterns
