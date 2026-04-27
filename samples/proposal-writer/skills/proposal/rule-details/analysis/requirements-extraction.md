# Requirements Extraction — Analysis Phase

## Purpose

This stage systematically extracts, categorizes, and organizes all requirements from the RFP document. The extracted requirements form the foundation for compliance tracking, proposal structure, and content planning.

## Prerequisites
- RFP Intake stage completed
- `proposal-docs/analysis/rfp-summary.md` exists and approved
- Common rules loaded: `common/content-validation.md`, `common/question-format-guide.md`

## Execution Classification
**Type**: ALWAYS
Requirements extraction always executes because compliance requires a complete understanding of all requirements.

## Adaptive Depth
- **Minimal**: Simple RFP with clearly numbered requirements — Direct extraction and categorization
- **Standard**: Moderate RFP with requirements spread across multiple sections — Cross-section analysis, categorization, and priority mapping
- **Comprehensive**: Complex RFP with implicit requirements, cross-references, and regulatory standards — Full extraction including implied requirements, regulatory decomposition, and traceability mapping

---

## Execution Steps

### Step 1: Explicit Requirements Extraction
**Action**: Identify all explicitly stated requirements in the RFP
**Input**: Full RFP document
**Output**: List of requirements with:
- Requirement ID (assigned or from RFP)
- Requirement text (verbatim from RFP)
- Source location (section number and page)
- Requirement type (functional, technical, management, administrative)
**Validation**: Every section of the RFP has been reviewed; no explicit requirements missed

### Step 2: Implicit Requirements Extraction
**Action**: Identify requirements implied by the RFP but not explicitly stated
**Input**: Full RFP document, evaluation criteria, industry standards referenced
**Output**: List of implicit requirements with:
- Derived requirement text
- Source (what implies this requirement)
- Confidence level (High / Medium / Low)
**Validation**: Implicit requirements are reasonable inferences, not assumptions (flag Low confidence for user review)

### Step 3: Requirements Categorization
**Action**: Classify each requirement by priority and type
**Input**: All extracted requirements (explicit and implicit)
**Output**: Categorized requirements:
- **Mandatory** (shall/must): Must comply or risk disqualification
- **Desirable** (should): Adds scoring value if addressed
- **Informational** (may/describe): Requested information without strict compliance standard
**Validation**: Classification aligns with RFP language; ambiguous language flagged for user decision

### Step 4: Evaluation Criteria Mapping
**Action**: Map requirements to evaluation criteria and scoring weights
**Input**: Requirements list, evaluation criteria from RFP summary
**Output**: Requirements-to-criteria mapping showing:
- Which requirements affect which evaluation criterion
- Relative importance based on scoring weights
- Critical requirements (mandatory + high-weight evaluation)
**Validation**: All evaluation criteria have associated requirements; highest-weight criteria have adequate requirement coverage

### Step 5: Submission Format Requirements
**Action**: Extract all proposal format and submission requirements
**Input**: RFP submission instructions section
**Output**: Format requirements including:
- Page limits (overall and per section, if specified)
- Font, margins, and formatting specifications
- Volume structure (if multi-volume required)
- Required sections and their order
- Attachments, forms, and certifications required
- Electronic vs. physical submission details
**Validation**: All format constraints documented; conflicts with content needs flagged

### Step 6: Compliance and Regulatory Requirements
**Action**: Identify regulatory, legal, and compliance requirements
**Input**: RFP terms and conditions, referenced standards and regulations
**Output**: Compliance requirements including:
- Referenced standards (ISO, NIST, CMMI, etc.)
- Regulatory compliance (HIPAA, FISMA, GDPR, etc.)
- Certification requirements
- Insurance and bonding requirements
- Small business / diversity requirements
**Validation**: All referenced standards identified; user confirmed which apply to their organization

### Step 7: Ambiguity and Conflict Identification
**Action**: Flag requirements that are ambiguous, conflicting, or incomplete
**Input**: All extracted requirements
**Output**: List of issues including:
- Ambiguous requirements (multiple valid interpretations)
- Conflicting requirements (two requirements that cannot both be fully met)
- Incomplete requirements (missing details needed for response)
- Recommendation for each (ask issuer, assume interpretation, request user decision)
**Validation**: All flagged items presented to user for resolution

### Step 8: Requirements Document Generation
**Action**: Create the comprehensive requirements document
**Input**: All outputs from Steps 1-7
**Output**: `proposal-docs/analysis/requirements.md` following the Requirements Document Pattern from `common/output-structure-patterns.md`
**Validation**: Document contains all extracted requirements with proper categorization and traceability

---

## Question Generation

### Question Categories
- **Classification**: Questions about whether a requirement is mandatory or desirable
- **Interpretation**: Questions about how to interpret ambiguous requirement language
- **Capability**: Questions about the user's ability to comply with specific requirements
- **Priority**: Questions about which requirements to emphasize when trade-offs are needed

### Question File Template

```markdown
# Requirements Extraction Questions

Please answer the following questions to help clarify the extracted requirements.

## Question 1
The RFP states "[ambiguous text]" in Section [X]. This could mean:

A) [Interpretation 1 — more restrictive]
B) [Interpretation 2 — less restrictive]
C) [Interpretation 3 — alternative reading]
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### Requirements Document
- **File**: `proposal-docs/analysis/requirements.md`
- **Content**: Complete categorized requirements with traceability to RFP sections
- **Format**: Follows the Requirements Document Pattern from `common/output-structure-patterns.md`

---

## Completion Message

### REVIEW REQUIRED

**Requirements Extraction is complete.** Here's what was produced:

- [N] total requirements extracted ([N] mandatory, [N] desirable, [N] informational)
- Requirements mapped to [N] evaluation criteria
- [N] ambiguities/conflicts identified for resolution
- Submission format requirements documented

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the requirements based on your feedback
**B) Continue to Compliance Matrix** — Proceed to compliance assessment

---

## Error Handling

**Error**: Cannot determine if requirement is mandatory or desirable
- **Solution**: Generate question file asking user to classify the requirement
- **Default**: Treat as mandatory for compliance safety

**Error**: RFP references external standard not included in the document
- **Solution**: Flag the reference, ask user if they have the standard document
- **Workaround**: Note as "referenced standard — compliance TBD" in requirements

**Error**: Requirements conflict with each other
- **Solution**: Document both requirements, flag conflict, ask user for resolution approach
- **Do Not Proceed**: With conflicting requirements unresolved in the compliance matrix

See `common/error-handling.md` for general error handling procedures.
