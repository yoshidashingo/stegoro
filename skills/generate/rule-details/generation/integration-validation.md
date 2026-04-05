# Integration Validation — GENERATION Phase

## Purpose
Validate that the complete generated steering policy set works as an integrated whole. This stage checks cross-references, workflow flow completeness, pattern consistency, and quality dimension coverage across all generated files.

## Prerequisites
- Core Workflow Generation completed and approved
- Common Rules Generation completed and approved
- Phase Rules Generation completed and approved
- All generated files written to disk

## Execution Classification
**Type**: ALWAYS
This stage always executes as it's the final quality gate before REFINEMENT.

---

## 3-Layer Testing Architecture

This stage executes 3 layers of testing. ALL layers must PASS.

| Layer | Purpose | Checks | Pass Criteria |
|-------|---------|--------|---------------|
| **Structure** | Verify file integrity | File existence, core-workflow reference resolution, Markdown syntax, flow path traversal (all stages reachable) | 0 errors |
| **Content** | Verify content quality | Dim 12: domain specificity >=40% per Phase Rule, Dim 13: examples >=2/file, Dim 14: artifact templates =100%, Dim 15: pitfall references >=50% | All above threshold |
| **Smoke** | Verify workflow execution | Virtually trace all stages in core-workflow for 4 agent types (Process/Task/Analytical/Hybrid), verify step→output→next stage chain | 0 blocking inconsistencies |

### FAIL Routing

| Layer FAIL | Classification | Return To |
|-----------|---------------|-----------|
| Structure FAIL | Structural issue | G1 (Core Workflow Generation) |
| Content FAIL (Dim 12-15) | Content issue | G3 (Phase Rules Generation) |
| Smoke FAIL (flow break) | Structural issue | G1 (Core Workflow Generation) |

### Validation Report Format

```markdown
## Integration Validation Report

### Structure Tests
- Total checks: [N] / PASS: [N] / FAIL: [N]

### Content Tests
- Dim 12 (Domain Specificity): [N]% (threshold: 40%) — [PASS/FAIL]
- Dim 13 (Example Coverage): [N] examples/file (threshold: 2) — [PASS/FAIL]
- Dim 14 (Artifact Templates): [N]% (threshold: 100%) — [PASS/FAIL]
- Dim 15 (Pitfall Reference): [N]% (threshold: 50%) — [PASS/FAIL]

### Smoke Tests
- Process: [PASS/FAIL] | Task: [PASS/FAIL] | Analytical: [PASS/FAIL] | Hybrid: [PASS/FAIL]

### Overall: [PASS/FAIL]
```

---

## Execution Steps

### Step 1: Inventory All Generated Files
**Action**: Create a complete inventory of all generated files
**Input**: Target agent directory
**Output**: File inventory with basic statistics

```markdown
# Generated File Inventory

| # | File Path | Lines | Size |
|---|-----------|-------|------|
| 1 | core-workflow.md | [N] | [size] |
| 2 | common/terminology.md | [N] | [size] |
| ... | ... | ... | ... |

**Total Files**: [N]
**Total Lines**: [N]
```

Verify:
- [ ] All files from the design inventory exist on disk
- [ ] No unexpected extra files
- [ ] All directories created as specified

### Step 2: File Reference Validation
**Action**: Verify all file references resolve correctly
**Input**: All generated files
**Output**: Reference validation report

#### Validation Process

1. **Extract all file references** from core-workflow.md
   - Identify every `Load all steps from...` reference
   - Identify every `Load...` reference
   - Identify every path reference

2. **Extract all cross-references** from common rule files
   - Internal references between common rules
   - References from common rules to phase rules
   - References to external resources

3. **Extract all cross-references** from phase rule files
   - References to common rules
   - References to other phase rules
   - References to core workflow
   - References within same phase

4. **Build reference graph**
   - Create a directed graph of all file references
   - Identify any broken references (pointing to non-existent files)
   - Identify any orphaned files (not referenced by any other file)

5. **Report findings**

```markdown
## File Reference Validation

### Core Workflow References
| Reference | Target | Status |
|-----------|--------|--------|
| `common/terminology.md` | common/terminology.md | RESOLVED |
| `[phase]/[stage].md` | [phase]/[stage].md | RESOLVED |

**Broken References**: [N]
**Orphaned Files**: [N]
```

### Step 3: Workflow Flow Validation
**Action**: Trace every possible workflow path to verify completeness
**Input**: Core workflow + all stage files
**Output**: Flow validation report

#### Validation Process

1. **Identify all entry points** (where the workflow starts)
2. **Trace each path** through the workflow:
   - Follow ALWAYS stages sequentially
   - For CONDITIONAL stages, trace both execute and skip paths
   - For two-part stages, trace Planning → Approval → Generation
3. **Verify each path reaches a valid end point**
4. **Check for dead ends** (stages with no defined next step)
5. **Check for unreachable stages** (stages no path leads to)
6. **Check for infinite loops** (circular paths without exit)

```markdown
## Flow Validation

### Workflow Paths Traced
| Path | Stages | Status |
|------|--------|--------|
| All ALWAYS | [list] | VALID |
| Skip [conditional] | [list] | VALID |
| Execute [conditional] | [list] | VALID |

**Dead Ends Found**: [N]
**Unreachable Stages**: [N]
**Circular Paths**: [N]
```

### Step 4: Pattern Consistency Validation
**Action**: Verify all files follow consistent patterns
**Input**: All generated files + output-structure-patterns.md
**Output**: Pattern consistency report

#### Pattern Checks

1. **Core Workflow Pattern**:
   - [ ] Has priority override statement
   - [ ] Has MANDATORY sections for rules, validation, questions, welcome
   - [ ] Each phase has header with purpose and focus
   - [ ] Each stage has numbered execution steps
   - [ ] Each stage has audit logging steps
   - [ ] Each stage has approval gate
   - [ ] Key principles section present
   - [ ] Checkbox enforcement rules present
   - [ ] Logging requirements present

2. **Common Rule File Pattern**:
   - [ ] Each file has Purpose section
   - [ ] Main content is organized with clear headings
   - [ ] Implementation guidelines included
   - [ ] Error handling section present
   - [ ] References section present

3. **Phase Rule File Pattern**:
   - [ ] Each file has Purpose section
   - [ ] Each file has Prerequisites section
   - [ ] Each file has Execution Classification
   - [ ] Each file has numbered Execution Steps
   - [ ] Each file has Question Generation section
   - [ ] Each file has Output Artifacts section
   - [ ] Each file has Completion Message with 2-option format
   - [ ] Each file has Error Handling section

4. **Completion Message Pattern**:
   - [ ] All completion messages use "REVIEW REQUIRED" heading
   - [ ] All completion messages have "WHAT'S NEXT" heading
   - [ ] All completion messages offer exactly 2 options
   - [ ] Option A is always "Request Changes"
   - [ ] Option B always references the correct next stage

```markdown
## Pattern Consistency

### Pattern Compliance by File Type
| File Type | Compliant | Non-Compliant | Details |
|-----------|-----------|---------------|---------|
| Core Workflow | [N]/[N] | [N] | [Issues] |
| Common Rules | [N]/[N] | [N] | [Issues] |
| Phase Rules | [N]/[N] | [N] | [Issues] |
| Completion Messages | [N]/[N] | [N] | [Issues] |
```

### Step 5: Terminology Consistency Validation
**Action**: Verify terminology usage matches terminology.md across all files
**Input**: terminology.md + all other files
**Output**: Terminology consistency report

Check:
- Phase names used consistently (exact match to terminology.md definitions)
- Stage names used consistently
- Agent type terminology consistent
- Execution classifications used consistently (ALWAYS, CONDITIONAL, not variations)
- Domain-specific terms match glossary definitions

```markdown
## Terminology Consistency

### Term Usage Audit
| Term | Definition (terminology.md) | Consistent? | Violations |
|------|-----------------------------|-------------|------------|
| [Phase Name] | [Definition] | Yes/No | [Files with violations] |
```

### Step 6: Quality Dimension Coverage Verification
**Action**: Verify all 15 quality dimensions are represented in generated files
**Input**: All generated files + quality-standards.md
**Output**: Quality dimension coverage report

For each of the 15 dimensions:
1. Which generated files implement this dimension?
2. Is the implementation complete or partial?
3. Is the implementation consistent across files?

```markdown
## Quality Dimension Coverage

| Dimension | Implemented In | Status | Notes |
|-----------|---------------|--------|-------|
| 1. Adaptive Workflow | core-workflow.md, [stages] | COVERED | |
| 2. Mandatory Checkpoints | core-workflow.md, [stages] | COVERED | |
| 3. Question File Format | question-format-guide.md, [stages] | COVERED | |
| 4. Content Validation | content-validation.md | COVERED | |
| 5. Audit Trail | core-workflow.md, [stages] | COVERED | |
| 6. Error Handling | error-handling.md, [stages] | COVERED | |
| 7. Overconfidence Prevention | overconfidence-prevention.md, [stages] | COVERED | |
| 8. Depth Levels | [stages with adaptive depth] | COVERED | |
| 9. Session Continuity | session-continuity.md | COVERED | |
| 10. Terminology | terminology.md | COVERED | |
| 11. Completion Messages | [all stage files] | COVERED | |

**Dimensions Fully Covered**: [N]/11
**Dimensions Partially Covered**: [N]/11
**Dimensions Missing**: [N]/11
```

### Step 7: Generate Integration Validation Report
**Action**: Compile all validation results into a comprehensive report
**Input**: All validation results
**Output**: Integration validation report

### Step 8: Handle Validation Failures
**Action**: If validation issues are found, determine next steps
**Input**: Validation report
**Output**: Fix plan or return to generation

**If issues found**:
1. Categorize issues by severity (blocking vs non-blocking)
2. Fix non-blocking issues in-place
3. For blocking issues, determine which generation stage to return to
4. Present fix plan to user

### Step 9: Present Validation Report
**Action**: Present the complete validation report for approval
**Input**: Validation report
**Output**: Formatted report for user review

---

## Output Artifacts

### Integration Validation Report
- **File**: `steering-docs/generation/integration-validation-report.md`
- **Content**: Complete validation results across all categories
- **Format**:

```markdown
# Integration Validation Report

## Executive Summary
- **Total Files**: [N]
- **Total Lines**: [N]
- **Reference Validation**: [PASS/FAIL] ([N] broken, [N] orphaned)
- **Flow Validation**: [PASS/FAIL] ([N] dead ends, [N] unreachable)
- **Pattern Consistency**: [PASS/FAIL] ([N] non-compliant)
- **Terminology Consistency**: [PASS/FAIL] ([N] violations)
- **Quality Dimensions**: [N]/15 fully covered

## Overall Status: [PASS / FAIL]

[Detailed sections for each validation category]
```

---

## Completion Message

### REVIEW REQUIRED

**Integration Validation is complete.** Here's the validation summary:

- **Total Files Validated**: [N] ([N] total lines)
- **File References**: [PASS/FAIL] — [N] references checked
- **Workflow Flow**: [PASS/FAIL] — [N] paths traced
- **Pattern Consistency**: [PASS/FAIL] — [N] files checked
- **Terminology**: [PASS/FAIL] — [N] terms verified
- **Quality Dimensions**: [N]/15 covered

**Overall Status**: [PASS / FAIL]

[If FAIL: List issues and proposed fixes]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll fix the identified issues before proceeding
**B) Continue to REFINEMENT Phase** — Proceed to quality review and calibration

---

### PHASE COMPLETE: GENERATION

**All stages in the GENERATION phase are complete.**

**Summary of outputs:**
- Core Workflow: `.<agent-name>/<agent-name>-rules/core-workflow.md` ([N] lines)
- Common Rules: [N] files in `common/` ([N] total lines)
- Phase Rules: [N] files across [N] phases ([N] total lines)
- Integration Validation: [PASS/FAIL]

**Total Generated**: [N] files, [N] lines

### WHAT'S NEXT

The next phase is **REFINEMENT**: Ensure AI-DLC-level quality.

**A) Proceed to REFINEMENT Phase** — Continue the workflow
**B) Review GENERATION outputs** — Review before moving forward

---

## Error Handling

### Many Broken References
- **Issue**: Significant number of file references don't resolve
- **Solution**: Likely a systematic issue — check directory structure and naming conventions
- **Recovery**: May need to regenerate files with corrected paths

### Dead-End Workflow Paths
- **Issue**: Some workflow paths lead to stages with no defined next step
- **Solution**: Update completion messages to reference correct next stages
- **Recovery**: Fix in phase rule files, re-validate

### Pattern Non-Compliance
- **Issue**: Files don't follow the established structural patterns
- **Solution**: Reformat non-compliant files to match output-structure-patterns.md
- **Recovery**: Systematic pattern update across affected files

### Quality Dimension Gaps
- **Issue**: One or more quality dimensions not represented in generated files
- **Solution**: Return to appropriate generation stage to add missing quality mechanisms
- **Recovery**: May need to create additional files or expand existing ones

## References
- `common/content-validation.md` — Validation rules
- `common/output-structure-patterns.md` — Expected patterns
- `common/terminology.md` — Terminology reference
- `common/quality-standards.md` — Quality dimension definitions
- `design/workflow-architecture.md` — Expected workflow structure
- `design/phase-rules-design.md` — Expected file inventory
