# Final Assembly — Review Phase

## Purpose

This stage compiles all approved proposal sections into the final submission package, generates the table of contents, creates the submission checklist, and produces a final proposal summary. This is the last stage before the proposal is ready for submission.

## Prerequisites
- Compliance Check completed (all critical gaps resolved)
- Quality Review completed (critical improvements addressed)
- `proposal-docs/review/compliance-check-report.md` — Final compliance status
- `proposal-docs/review/quality-review-report.md` — Final quality status
- All approved proposal sections:
  - `proposal-docs/proposal/executive-summary.md`
  - `proposal-docs/proposal/technical-response.md`
  - `proposal-docs/proposal/management-response.md` (if executed)
  - `proposal-docs/proposal/pricing-framework.md` (if executed)
- Common rules loaded: `common/content-validation.md`, `common/output-structure-patterns.md`

## Execution Classification
**Type**: ALWAYS
Final assembly always executes because every proposal must be compiled into a cohesive submission package.

---

## Execution Steps

### Step 1: Final Content Inventory
**Action**: Verify all required proposal components are present and approved
**Input**: Proposal structure document, all written sections
**Output**: Content inventory:
- List of all sections with status (Present / Missing / Draft)
- Page count per section
- Total page count vs. limit
- All required attachments and forms identified
**Validation**: All required sections are present; no section is in "Draft" status; page count is within limit

### Step 2: Table of Contents Generation
**Action**: Generate the final table of contents with accurate section references
**Input**: All proposal sections in final order
**Output**: Complete table of contents with:
- All section and subsection headings
- Page references (estimated based on content length)
- Figure and table lists (if applicable)
**Validation**: Every heading in the proposal appears in the TOC; page references are sequential and reasonable

### Step 3: Cross-Reference Verification
**Action**: Verify all internal cross-references resolve correctly
**Input**: All proposal sections
**Output**: Cross-reference verification results:
- All "See Section X" references point to existing sections
- Compliance matrix section references are accurate
- Figure and table references are correct
- No broken or circular references
**Validation**: 100% of cross-references resolve; no orphan references

### Step 4: Final Formatting Check
**Action**: Verify consistent formatting across all sections
**Input**: All proposal sections
**Output**: Formatting verification:
- Heading styles consistent across sections
- List formatting consistent (bullets, numbers)
- Table formatting consistent
- Consistent terminology and abbreviation usage
- No placeholder text remaining ("[TBD]", "[INSERT]", "[USER INPUT]")
**Validation**: No formatting inconsistencies; no placeholder text (or clearly marked as user-dependent fields)

### Step 5: Submission Checklist Generation
**Action**: Create the submission checklist for final verification before delivery
**Input**: RFP submission requirements, content inventory
**Output**: `proposal-docs/review/submission-checklist.md` following the Submission Checklist Pattern from `common/output-structure-patterns.md`
**Validation**: Checklist covers all RFP submission requirements; no requirements missed

### Step 6: Final Proposal Summary
**Action**: Create a summary of the complete proposal for reference
**Input**: All proposal components, compliance check report, quality review report
**Output**: Final summary including:
- Proposal statistics (sections, pages, requirements addressed)
- Compliance score (% of requirements fully addressed)
- Quality rating (from quality review)
- Key win themes
- Conditional stages executed/skipped
- Outstanding items for user attention
**Validation**: Summary accurately reflects the final proposal state

### Step 7: Update Final State
**Action**: Update proposal-state.md to reflect completion
**Input**: All completion data
**Output**: Updated `proposal-docs/proposal-state.md` with all stages marked complete
**Validation**: State file accurately reflects workflow completion

---

## Output Artifacts

### Submission Checklist
- **File**: `proposal-docs/review/submission-checklist.md`
- **Content**: Comprehensive checklist for pre-submission verification
- **Format**: Following the Submission Checklist Pattern from `common/output-structure-patterns.md`

### Final Proposal Summary
- Included in the completion message (not a separate file)

---

## Completion Message

### PROPOSAL COMPLETE

**The proposal for [RFP Title] has been generated and reviewed.**

**Final Deliverables:**
- Directory: `proposal-docs/`
- Total sections: [N]
- Total estimated pages: [N] (limit: [N])
- Compliance: [N]/[N] mandatory requirements addressed ([%])
- Quality rating: [Rating]

**Contents:**
| File | Section | Pages | Status |
|------|---------|-------|--------|
| `proposal/executive-summary.md` | Executive Summary | [N] | Complete |
| `proposal/technical-response.md` | Technical Response | [N] | Complete |
| `proposal/management-response.md` | Management Response | [N] | Complete / Skipped |
| `proposal/pricing-framework.md` | Pricing Framework | [N] | Complete / Skipped |
| `review/compliance-check-report.md` | Compliance Report | — | Complete |
| `review/quality-review-report.md` | Quality Report | — | Complete |
| `review/submission-checklist.md` | Submission Checklist | — | Complete |

**Win Themes:**
1. [Theme 1]
2. [Theme 2]
3. [Theme 3]

### NEXT STEPS

1. Review the complete proposal in `proposal-docs/`
2. Fill in any remaining user-dependent fields (pricing data, specific names)
3. Conduct internal review with subject matter experts
4. Transfer content to required submission format/templates (if needed)
5. Complete the submission checklist
6. Submit before the deadline: [Deadline]

---

## Error Handling

**Error**: Required sections are missing or incomplete
- **Solution**: Identify missing sections; return to the appropriate Writing phase stage
- **Do Not Assemble**: An incomplete proposal (all required sections must be present)

**Error**: Page count exceeds limit
- **Solution**: Present page overrun by section; recommend trimming priorities
- **Recovery**: Reduce lowest-priority content; condense redundant material; consider appendices

**Error**: Placeholder text remains in the proposal
- **Solution**: List all remaining placeholders with their locations
- **Note**: Some placeholders may be intentional (user-dependent fields like pricing); differentiate between errors and intentional blanks

**Error**: Cross-references are broken after final edits
- **Solution**: Re-run cross-reference verification; fix broken references
- **Recovery**: Update section references to reflect final structure

See `common/error-handling.md` for general error handling procedures.
