# Compliance Matrix — Analysis Phase

## Purpose

This stage creates a comprehensive compliance matrix that tracks every RFP requirement against the organization's ability to comply. The compliance matrix is the central tracking document throughout the entire proposal process, updated at each subsequent stage.

## Prerequisites
- RFP Intake and Requirements Extraction stages completed
- `proposal-docs/analysis/rfp-summary.md` exists and approved
- `proposal-docs/analysis/requirements.md` exists and approved
- Common rules loaded: `common/content-validation.md`, `common/question-format-guide.md`

## Execution Classification
**Type**: ALWAYS
Compliance matrix always executes because every proposal must demonstrate compliance with RFP requirements.

## Adaptive Depth
- **Minimal**: Simple RFP with few requirements — Basic compliance/non-compliance status
- **Standard**: Moderate RFP — Full compliance status with mitigation notes for gaps
- **Comprehensive**: Complex RFP — Detailed compliance analysis with risk scoring, mitigation strategies, and evidence mapping

---

## Execution Steps

### Step 1: Initialize Compliance Matrix
**Action**: Create the compliance matrix structure with all extracted requirements
**Input**: `proposal-docs/analysis/requirements.md`
**Output**: Initial matrix with all requirement IDs, descriptions, and empty status fields
**Validation**: Every requirement from the requirements document appears in the matrix

### Step 2: Compliance Assessment
**Action**: Assess initial compliance status for each requirement
**Input**: Requirements list, user-provided capability information
**Output**: For each requirement, one of:
- **Compliant**: Can fully meet the requirement
- **Partially Compliant**: Can meet with limitations, alternatives, or conditions
- **Non-Compliant**: Cannot meet the requirement
- **Not Applicable**: Requirement does not apply to this response
- **TBD**: Compliance cannot be determined without additional information
**Validation**: No requirement left unassessed; TBD items have specific follow-up questions

### Step 3: Gap Analysis
**Action**: Analyze compliance gaps (Partial, Non-Compliant, TBD items)
**Input**: Compliance assessment from Step 2
**Output**: For each gap:
- Impact assessment (how much does this affect evaluation scoring?)
- Risk level (High / Medium / Low)
- Root cause (capability gap, information gap, scope gap)
**Validation**: All gaps have impact and risk assessments

### Step 4: Mitigation Strategy Development
**Action**: Develop mitigation strategies for compliance gaps
**Input**: Gap analysis from Step 3
**Output**: For each gap, one or more of:
- **Alternative approach**: Different way to meet the requirement's intent
- **Exception with rationale**: Formal exception with explanation of why and alternative offered
- **Development plan**: Commitment to develop the capability with timeline
- **Teaming strategy**: Partner with another organization that has the capability
- **Risk acceptance**: Acknowledge the gap and provide best available response
**Validation**: Every Partial and Non-Compliant requirement has at least one mitigation strategy

### Step 5: Proposal Section Mapping
**Action**: Map each requirement to the proposal section where it will be addressed
**Input**: Compliance matrix, draft proposal structure (from RFP's suggested format)
**Output**: Updated matrix with:
- Target proposal section for each requirement
- Requirements grouped by proposal section
- Cross-reference plan for requirements spanning multiple sections
**Validation**: Every Compliant and Partially Compliant requirement maps to at least one proposal section

### Step 6: Risk Scoring
**Action**: Calculate an overall compliance risk score
**Input**: Complete compliance matrix with assessments and mitigations
**Output**: Risk summary including:
- Overall compliance percentage
- High-risk requirement count
- Critical gaps (mandatory requirements with Non-Compliant or TBD status)
- Risk score by evaluation criterion
**Validation**: Risk scores align with gap analysis; critical gaps are prominently flagged

### Step 7: Compliance Matrix Document Generation
**Action**: Generate the final compliance matrix document
**Input**: All outputs from Steps 1-6
**Output**: `proposal-docs/analysis/compliance-matrix.md` following the Compliance Matrix Pattern from `common/output-structure-patterns.md`
**Validation**: Matrix is complete, internally consistent, and cross-references are accurate

---

## Question Generation

### Question Categories
- **Capability**: Questions about whether the organization can meet specific requirements
- **Mitigation**: Questions about preferred approach for compliance gaps
- **Prioritization**: Questions about which gaps to address first given limited resources
- **Exception**: Questions about whether to take exception to specific requirements

### Question File Template

```markdown
# Compliance Matrix Questions

Please answer the following questions to help assess compliance with RFP requirements.

## Question 1
Requirement [ID] requires "[requirement description]". Can your organization fully comply?

A) Yes, fully compliant — We meet this requirement completely
B) Partially — We can meet this with the following limitations: [describe after Answer tag]
C) No — We cannot meet this requirement
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### Compliance Matrix
- **File**: `proposal-docs/analysis/compliance-matrix.md`
- **Content**: Complete requirement-by-requirement compliance status with mitigation strategies and proposal section mapping
- **Format**: Follows the Compliance Matrix Pattern from `common/output-structure-patterns.md`

---

## Completion Message

### REVIEW REQUIRED

**Compliance Matrix is complete.** Here's what was produced:

- [N] requirements assessed: [N] Compliant, [N] Partial, [N] Non-Compliant, [N] TBD
- Overall compliance: [X]%
- [N] critical gaps identified with mitigation strategies
- Requirements mapped to [N] proposal sections

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the compliance matrix based on your feedback
**B) Continue to Proposal Structure Design** — Proceed to the Planning phase

---

## Error Handling

**Error**: Cannot determine compliance without additional capability information
- **Solution**: Generate question file targeting specific capability gaps
- **Workaround**: Mark as TBD and track for resolution in Planning phase

**Error**: Mandatory requirements are Non-Compliant with no viable mitigation
- **Solution**: Discuss go/no-go decision with user — may indicate the RFP is not a good fit
- **Escalation**: This is a business decision requiring user judgment

**Error**: Requirements are too vague to assess compliance
- **Solution**: Document as TBD, recommend user contact issuing organization for clarification
- **Workaround**: Assess against most likely interpretation, flag as assumption

See `common/error-handling.md` for general error handling procedures.
