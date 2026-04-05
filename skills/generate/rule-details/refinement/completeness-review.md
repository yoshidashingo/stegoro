# Completeness Review — REFINEMENT Phase

## Purpose
Verify that the generated steering policy set is complete — all workflow paths have corresponding rule files, all required sections are present, all quality mechanisms are implemented, and no gaps exist that would cause the target agent to fail during operation.

## Prerequisites
- All GENERATION phase artifacts available and approved
- Integration Validation completed and passed
- All generated policy files written to disk
- Access to all DISCOVERY and DESIGN phase artifacts for verification

## Execution Classification
**Type**: ALWAYS
This stage always executes as completeness is fundamental to a usable steering policy set.

---

## Execution Steps

### Step 1: Load Complete Context
**Action**: Load all generated files and design specifications
**Input**: All generated policy files + all design artifacts
**Output**: Complete context for review

Load:
- All generated files (core workflow + common rules + phase rules)
- Workflow architecture (expected structure)
- Phase rules design (expected file inventory)
- Quality mechanisms design (expected quality features)
- Integration validation report (known issues)

### Step 2: Workflow Path Completeness Check
**Action**: Verify every workflow path has complete rule file coverage
**Input**: Core workflow + all phase rule files
**Output**: Workflow path completeness report

#### Verification Process

1. **Extract all workflow paths** from core-workflow.md
2. For each path:
   - List all stages on the path
   - Verify each stage has a corresponding rule file
   - Verify each rule file has complete content (not placeholder)
   - Verify the path has clear start and end points
3. **Special attention to CONDITIONAL paths**:
   - Both "execute" and "skip" paths must be handled
   - Skip paths must still lead to valid next stages

```markdown
## Workflow Path Completeness

### Path 1: All ALWAYS stages
| Stage | Rule File | Complete? | Notes |
|-------|-----------|-----------|-------|
| [Stage 1] | [file].md | Yes/No | |
| [Stage 2] | [file].md | Yes/No | |

### Path 2: With [Conditional Stage] executed
[Similar table]

### Path 3: With [Conditional Stage] skipped
[Similar table]

**Complete Paths**: [N]/[N]
**Incomplete Paths**: [N] (details below)
```

### Step 3: Stage File Section Completeness Check
**Action**: Verify each phase rule file has all required sections
**Input**: All phase rule files + output-structure-patterns.md
**Output**: Section completeness report

#### Required Sections for Each Phase Rule File

- [ ] **Purpose**: Clear statement of what the stage achieves
- [ ] **Prerequisites**: List of required prior artifacts and loaded rules
- [ ] **Execution Classification**: ALWAYS or CONDITIONAL with criteria
- [ ] **Execution Steps**: Numbered steps with Action/Input/Output/Validation
- [ ] **Question Generation**: When and what questions to ask
- [ ] **Output Artifacts**: What the stage produces with format descriptions
- [ ] **Completion Message**: REVIEW REQUIRED + WHAT'S NEXT with 2 options
- [ ] **Error Handling**: Stage-specific error scenarios and recovery

#### Required Sections for Each Common Rule File

- [ ] **Purpose**: Clear explanation of what the rule governs
- [ ] **Main Content**: Organized content with clear headings
- [ ] **Implementation Guidelines**: How to apply the rule
- [ ] **Error Handling**: What to do when the rule can't be followed

### Step 4: ALWAYS/CONDITIONAL Classification Completeness
**Action**: Verify all stages have explicit execution classifications
**Input**: Core workflow + all stage files
**Output**: Classification completeness report

For every stage:
- [ ] Classification explicitly stated (ALWAYS or CONDITIONAL)
- [ ] For CONDITIONAL: "Execute IF" criteria present with specific conditions
- [ ] For CONDITIONAL: "Skip IF" criteria present with specific conditions
- [ ] Classification in stage file matches classification in core-workflow.md

### Step 5: Approval Gate Completeness
**Action**: Verify all approval gates are properly defined
**Input**: Core workflow + all stage files
**Output**: Approval gate completeness report

For every stage with an approval gate:
- [ ] "Wait for Explicit Approval" instruction in core-workflow.md
- [ ] Completion message in stage file with REVIEW REQUIRED heading
- [ ] Completion message has WHAT'S NEXT with exactly 2 options
- [ ] Option A is "Request Changes"
- [ ] Option B references the correct next stage
- [ ] MANDATORY audit logging before and after approval

### Step 6: Error Handling Completeness
**Action**: Verify error handling coverage across all files
**Input**: All generated files + quality mechanisms design
**Output**: Error handling completeness report

Verify:
- [ ] Common `error-handling.md` exists with all phases covered
- [ ] Each phase rule file has stage-specific error handling section
- [ ] Error severity levels defined (Critical/High/Medium/Low)
- [ ] Recovery procedures defined for each error type
- [ ] Escalation guidelines present
- [ ] Session resumption error handling included

### Step 7: Quality Dimension Completeness
**Action**: Verify all 15 quality dimensions are fully implemented
**Input**: All generated files + quality-standards.md
**Output**: Quality dimension completeness report

For each of the 15 dimensions, verify:

| Dimension | File(s) Implementing | Status | Missing Elements |
|-----------|---------------------|--------|-----------------|
| 1. Adaptive Workflow | core-workflow.md + stages | | |
| 2. Mandatory Checkpoints | core-workflow.md + stages | | |
| 3. Question File Format | question-format-guide.md + stages | | |
| 4. Content Validation | content-validation.md | | |
| 5. Audit Trail | core-workflow.md + stages | | |
| 6. Error Handling | error-handling.md + stages | | |
| 7. Overconfidence Prevention | overconfidence-prevention.md + stages | | |
| 8. Depth Levels | stages with adaptive depth | | |
| 9. Session Continuity | session-continuity.md | | |
| 10. Terminology | terminology.md | | |
| 11. Completion Messages | all stage files | | |

### Step 8: Quantitative Completeness Check
**Action**: Verify quantitative benchmarks are met
**Input**: All generated files
**Output**: Quantitative completeness report

Verify against scope definition targets:
- [ ] Total file count matches or exceeds estimate
- [ ] Total line count meets minimum threshold
- [ ] No phase rule file is under 80 lines (likely incomplete)
- [ ] No common rule file is under 40 lines (likely placeholder)
- [ ] Core workflow is between 200-500 lines
- [ ] All phase directories have at least 2 files

### Step 9: Generate Completeness Report
**Action**: Compile all completeness checks into a comprehensive report
**Input**: All check results
**Output**: Completeness review report

### Step 10: Handle Completeness Gaps
**Action**: If gaps are found, plan remediation
**Input**: Completeness report with gaps
**Output**: Remediation plan or return to GENERATION

**If gaps found**:
1. Categorize gaps by severity
2. Determine which generation stage can fix each gap
3. Estimate effort to fix
4. Present remediation plan to user
5. If minor: Fix in-place during REFINEMENT
6. If major: Return to GENERATION phase

### Step 11: Present Completeness Report
**Action**: Present the completeness review for approval
**Input**: Completeness report + remediation plan (if any)
**Output**: Formatted report for user review

---

## Output Artifacts

### Completeness Review Report
- **File**: `steering-docs/refinement/completeness-review-report.md`
- **Content**: Complete results of all completeness checks
- **Format**:

```markdown
# Completeness Review Report

## Summary
| Check | Status | Details |
|-------|--------|---------|
| Workflow Paths | PASS/FAIL | [N]/[N] complete |
| Section Completeness | PASS/FAIL | [N]/[N] compliant |
| Classifications | PASS/FAIL | [N]/[N] explicit |
| Approval Gates | PASS/FAIL | [N]/[N] defined |
| Error Handling | PASS/FAIL | [N]/[N] covered |
| Quality Dimensions | PASS/FAIL | [N]/15 complete |
| Quantitative | PASS/FAIL | [N] files, [N] lines |

## Overall Status: [PASS / FAIL]

[Detailed sections for each check]

## Gaps Found
[If any, with remediation plan]
```

---

## Content Depth Verification

In addition to structural completeness, verify content depth across all generated files:

### Content Depth Checklist

- [ ] All Phase Rule files have Purpose, Prerequisites, Steps, Error Handling, Completion Message sections
- [ ] All Phase Rule files have >= 2 GOOD/BAD examples (Dim 13 pre-check)
- [ ] All GENERATION files have artifact output templates (Dim 14 pre-check)
- [ ] All Error Handling sections contain domain-specific scenarios (not just generic errors)
- [ ] All Completion Messages use REVIEW REQUIRED + WHAT'S NEXT 2-option format

### Gap Classification and Routing

When gaps are found, classify and route via the repair judgment tree:

| Gap Type | Classification | Return To |
|----------|---------------|-----------|
| File missing / flow path break | Structural | G1 (Core Workflow Generation) |
| Section missing in existing file | Content | G3 (Phase Rules Generation) |
| Template missing in output file | Content | G2 or G3 |
| Examples insufficient (< 2/file) | Content | G3 (Phase Rules Generation) |
| Domain-specific content absent | Content | G3 (Phase Rules Generation) |

### Content Depth Report Format

```markdown
## Content Depth Summary

| File | Sections | Examples | Templates | Domain Errors | Status |
|------|----------|----------|-----------|---------------|--------|
| [file] | [N/5] | [N] | [Yes/No] | [Yes/No] | [OK/GAP] |
```

---

## Completion Message

### REVIEW REQUIRED

**Completeness Review is complete.** Here's the summary:

- **Workflow Paths**: [N]/[N] complete
- **Section Completeness**: [N]/[N] compliant
- **Classifications**: [N]/[N] explicit
- **Approval Gates**: [N]/[N] defined
- **Error Handling**: [N] phases covered
- **Quality Dimensions**: [N]/15 fully implemented
- **Quantitative**: [N] files, [N] lines

**Overall Status**: [PASS / FAIL]

[If FAIL: Summary of gaps and remediation plan]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll address the completeness gaps before proceeding
**B) Continue to Consistency Review** — Proceed to check internal consistency

---

## Error Handling

### Major Completeness Gaps
- **Issue**: Multiple workflow paths lack rule file coverage
- **Solution**: Return to Phase Rules Generation to create missing files
- **Do Not Proceed**: Until all workflow paths are covered

### Missing Quality Dimensions
- **Issue**: One or more quality dimensions not implemented
- **Solution**: Return to appropriate generation stage to add missing mechanisms
- **Recovery**: May require updates to multiple files

### Quantitative Shortfall
- **Issue**: Generated files don't meet minimum line count or file count
- **Solution**: Expand thin files with more domain-specific content
- **Recovery**: Focus on ALWAYS stage files first, then CONDITIONAL

## References
- `generation/integration-validation.md` — Prior validation results
- `design/phase-rules-design.md` — Expected file inventory
- `design/quality-mechanisms-design.md` — Expected quality features
- `common/quality-standards.md` — Quality dimension definitions
- `common/output-structure-patterns.md` — Expected file patterns
