# RFP Intake — Analysis Phase

## Purpose

This stage reads and parses the full RFP document, establishing a foundational understanding of what is being requested, who is requesting it, and what constraints govern the response.

## Prerequisites
- RFP document provided by user (any format: PDF, DOCX, Markdown, plain text, or pasted content)
- Common rules loaded: `common/process-overview.md`, `common/content-validation.md`

## Execution Classification
**Type**: ALWAYS
This stage always executes because every proposal requires understanding the source RFP.

## Adaptive Depth
- **Minimal**: Short, well-structured RFP (< 10 pages) — Quick summary and key dates
- **Standard**: Medium RFP (10-50 pages) — Full section-by-section analysis
- **Comprehensive**: Large RFP (50+ pages, multiple volumes/attachments) — Detailed analysis with attachment inventory

---

## Execution Steps

### Step 1: Document Reception
**Action**: Receive and confirm the RFP document from the user
**Input**: User-provided RFP document(s)
**Output**: Confirmation of received document(s), format, and page count
**Validation**: Verify the document is readable and complete (not truncated or corrupted)

### Step 2: Structure Analysis
**Action**: Identify the RFP's organizational structure
**Input**: Full RFP document
**Output**: Section inventory with titles, page ranges, and content descriptions
**Validation**: All sections identified; no orphan content between sections

### Step 3: Metadata Extraction
**Action**: Extract administrative and logistical information
**Input**: RFP document (cover page, administrative sections)
**Output**: Key metadata including:
- Issuing organization and contact information
- RFP number/identifier
- Issue date and submission deadline
- Q&A period and deadline (if applicable)
- Pre-proposal conference date (if applicable)
- Contract type (FFP, T&M, CPFF, etc.)
- Estimated contract value (if disclosed)
- Period of performance
- Small business / set-aside requirements (if applicable)
**Validation**: All available metadata fields populated; missing fields flagged

### Step 4: Scope Identification
**Action**: Determine the scope and nature of the procurement
**Input**: Statement of Work / Statement of Objectives / Performance Work Statement sections
**Output**: High-level scope summary including:
- What is being procured (products, services, or both)
- Primary domain (IT, consulting, construction, etc.)
- Geographic scope and location requirements
- Estimated team size or resource requirements
**Validation**: Scope aligns with all referenced sections; no conflicting scope statements

### Step 5: Evaluation Framework Identification
**Action**: Extract the evaluation methodology and criteria
**Input**: Evaluation criteria section of RFP
**Output**: Evaluation framework including:
- Evaluation factors and subfactors
- Scoring weights or relative importance
- Evaluation methodology (best value, LPTA, trade-off, etc.)
- Oral presentation requirements (if any)
**Validation**: Weights sum to 100% (if provided); all evaluation factors identified

### Step 6: Competitive Context Assessment
**Action**: Determine the competitive landscape
**Input**: RFP metadata, user knowledge, procurement context
**Output**: Assessment of:
- Sole source vs. competitive bid
- Incumbent contractor (if known)
- Likely number of competitors
- Re-compete vs. new requirement
**Validation**: Assessment logged; user confirmation requested if assumptions made

### Step 7: RFP Summary Generation
**Action**: Create the RFP summary document
**Input**: All outputs from Steps 1-6
**Output**: `proposal-docs/analysis/rfp-summary.md` following the RFP Summary Pattern from `common/output-structure-patterns.md`
**Validation**: Summary covers all extracted information; cross-references are accurate

---

## Question Generation

If the RFP is ambiguous or incomplete, generate questions in this stage:

### Question Categories
- **Scope Clarification**: Questions about what's in/out of scope
- **Format Clarification**: Questions about submission format requirements
- **Evaluation Clarification**: Questions about scoring methodology
- **Competitive Context**: Questions about known competitors or incumbent

### Question File Template

```markdown
# RFP Intake Questions

Please answer the following questions to help clarify the RFP context.

## Question 1
[Question about ambiguous RFP element]

A) [Option based on one interpretation]
B) [Option based on another interpretation]
C) [Option for user to provide additional context]
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### RFP Summary
- **File**: `proposal-docs/analysis/rfp-summary.md`
- **Content**: Complete summary of RFP metadata, scope, structure, evaluation framework, and competitive context
- **Format**: Follows the RFP Summary Pattern from `common/output-structure-patterns.md`

---

## Completion Message

### REVIEW REQUIRED

**RFP Intake is complete.** Here's what was produced:

- RFP summary document with structure analysis
- Key dates and submission requirements extracted
- Evaluation criteria and scoring methodology identified
- Competitive context assessed

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the RFP summary based on your feedback
**B) Continue to Requirements Extraction** — Proceed to detailed requirement analysis

---

## Error Handling

**Error**: RFP document cannot be parsed
- **Solution**: Ask user to provide in a different format or paste key sections directly
- **Workaround**: Proceed with verbal description from user

**Error**: RFP is missing key sections (no evaluation criteria, no submission deadline)
- **Solution**: Flag missing sections, ask user to check with issuing organization
- **Workaround**: Proceed with available information, flag gaps in compliance matrix

**Error**: Multiple conflicting versions of RFP provided
- **Solution**: Ask user to confirm which version is current
- **Do Not Proceed**: Until correct version is confirmed

See `common/error-handling.md` for general error handling procedures.
