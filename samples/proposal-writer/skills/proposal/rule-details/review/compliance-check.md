# Compliance Check — Review Phase

## Purpose

This stage systematically verifies that every RFP requirement is addressed in the proposal. The compliance check is a line-by-line audit of the compliance matrix against actual proposal content, ensuring no requirement was missed during writing and all compliance claims are substantiated.

## Prerequisites
- All previous phases completed (Analysis, Planning, Writing)
- All written proposal sections available:
  - `proposal-docs/proposal/executive-summary.md`
  - `proposal-docs/proposal/technical-response.md`
  - `proposal-docs/proposal/management-response.md` (if executed)
  - `proposal-docs/proposal/pricing-framework.md` (if executed)
- Original compliance matrix: `proposal-docs/analysis/compliance-matrix.md`
- Common rules loaded: `common/content-validation.md`

## Execution Classification
**Type**: ALWAYS
Compliance check always executes because submitting a non-compliant proposal wastes the entire effort.

---

## Execution Steps

### Step 1: Load and Cross-Reference
**Action**: Load the compliance matrix and all proposal sections
**Input**: Compliance matrix, all written proposal sections
**Output**: Cross-reference status for each requirement:
- Requirement ID
- Claimed compliance status (from matrix)
- Actual proposal section and paragraph where addressed
- Verified status (Yes / No / Partial)
**Validation**: Every requirement in the matrix is checked against actual proposal content

### Step 2: Mandatory Requirements Audit
**Action**: Verify all mandatory requirements are fully addressed
**Input**: Cross-reference from Step 1, requirements document (mandatory items)
**Output**: For each mandatory requirement:
- Is it addressed? (Yes / No / Partial)
- Where is it addressed? (Specific section and paragraph reference)
- Is the response sufficient? (Fully compliant / Partially compliant / Insufficient)
- Issues identified (missing detail, misalignment, contradiction)
**Validation**: Zero mandatory requirements should be unaddressed; any gaps are critical findings

### Step 3: Desirable Requirements Audit
**Action**: Verify desirable requirements are addressed to maximize scoring
**Input**: Cross-reference from Step 1, requirements document (desirable items)
**Output**: For each desirable requirement:
- Is it addressed? (Yes / No)
- If yes: strength of response (Strong / Adequate / Weak)
- If no: estimated scoring impact
**Validation**: High-weight desirable requirements should be addressed; low-weight items assessed for cost/benefit of inclusion

### Step 4: Submission Format Verification
**Action**: Verify the proposal meets all format and submission requirements
**Input**: Submission requirements from requirements document, actual proposal documents
**Output**: Format compliance checklist:
- Page count vs. page limit
- Required sections present and in correct order
- Font, margins, and formatting compliance (as applicable)
- All required forms and certifications included
- Volume structure matches requirements
**Validation**: All format requirements met; deviations flagged as critical findings

### Step 5: Cross-Section Consistency Check
**Action**: Verify consistency across all proposal sections
**Input**: All proposal sections
**Output**: Consistency findings:
- Company name and branding consistent
- Team member names and titles consistent across sections
- Technical terms and product names consistent
- Timeline dates consistent between technical and management sections
- Pricing structure aligns with technical scope
**Validation**: No contradictions between sections; terminology consistent throughout

### Step 6: Generate Compliance Check Report
**Action**: Create the compliance verification report
**Input**: All findings from Steps 1-5
**Output**: `proposal-docs/review/compliance-check-report.md`
**Validation**: Report is complete, accurately reflects findings, and clearly identifies action items

---

## Output Artifacts

### Compliance Check Report
- **File**: `proposal-docs/review/compliance-check-report.md`
- **Content**: Complete compliance verification results with findings and action items
- **Format**:

```markdown
# Compliance Check Report — [RFP Title]

## Summary
- **Total Requirements Checked**: [N]
- **Fully Addressed**: [N] ([%])
- **Partially Addressed**: [N] ([%])
- **Not Addressed**: [N] ([%])
- **Format Compliance**: [PASS / FAIL]
- **Cross-Section Consistency**: [PASS / FAIL — N issues found]

## Critical Findings (Must Fix)
| # | Requirement | Issue | Recommended Action |
|---|-------------|-------|-------------------|
| 1 | [Req ID] | [Description of gap] | [Specific fix needed] |

## Warnings (Should Fix)
| # | Requirement | Issue | Recommended Action |
|---|-------------|-------|-------------------|
| 1 | [Req ID] | [Description] | [Recommendation] |

## Detailed Results

### Mandatory Requirements
| Req ID | Status | Proposal Location | Assessment |
|--------|--------|------------------|------------|
| [ID] | [Addressed/Partial/Missing] | [Section ref] | [Notes] |

### Desirable Requirements
[Same format]

### Format Compliance
[Checklist results]

### Cross-Section Consistency
[Findings]

## Overall Compliance Status: [PASS / CONDITIONAL PASS / FAIL]
```

---

## Completion Message

### REVIEW REQUIRED

**Compliance Check is complete.** Here's a summary:

- [N] requirements verified: [N] fully addressed, [N] partial, [N] missing
- [N] critical findings requiring immediate action
- [N] warnings recommended for improvement
- Format compliance: [PASS/FAIL]
- Cross-section consistency: [N] issues found

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll address the compliance findings before proceeding
**B) Continue to Quality Review** — Proceed to content quality assessment

---

## Error Handling

**Error**: Critical compliance gaps found (mandatory requirements not addressed)
- **Solution**: Return to Writing phase to address specific gaps; present gap details with recommended content
- **Do Not Proceed**: To Final Assembly with unresolved critical gaps

**Error**: Format requirements violated (over page limit, wrong structure)
- **Solution**: Present specific violations and propose content reductions or restructuring
- **Recovery**: Trim lowest-priority content or reformat to meet requirements

**Error**: Cross-section contradictions found
- **Solution**: Present contradictions and ask user which version is correct
- **Recovery**: Update affected sections to maintain consistency

See `common/error-handling.md` for general error handling procedures.
