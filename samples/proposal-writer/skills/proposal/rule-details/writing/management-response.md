# Management Response — Writing Phase

## Purpose

This stage creates the management response section, which describes WHO will perform the work and HOW the project will be managed. The management response demonstrates organizational capability, team qualifications, and project management rigor.

## Prerequisites
- Analysis and Planning phases completed
- Executive Summary and Technical Response completed
- All prior artifacts loaded:
  - `proposal-docs/analysis/requirements.md` (management/staffing requirements)
  - `proposal-docs/analysis/compliance-matrix.md` (management compliance status)
  - `proposal-docs/planning/proposal-structure.md` (management section structure)
  - `proposal-docs/planning/content-plan.md` (management section outlines)
  - `proposal-docs/proposal/executive-summary.md` (for consistency)
  - `proposal-docs/proposal/technical-response.md` (for alignment with technical approach)
- Common rules loaded: `common/content-validation.md`, `common/overconfidence-prevention.md`

## Execution Classification
**Type**: CONDITIONAL

**Execute IF**:
- RFP requests management or organizational approach section
- Evaluation criteria include management/staffing/organizational scoring
- Team structure or key personnel section required
- Project management methodology evaluation included
- Past performance or corporate experience section required

**Skip IF**:
- RFP does not request management information
- Evaluation is purely technical and price-based
- User explicitly skips this section

---

## Execution Steps

### Step 1: Identify Management Response Requirements
**Action**: Determine exactly what management information the RFP requires
**Input**: Requirements document (management-related requirements), evaluation criteria
**Output**: Management response requirements including:
- Required subsections (org structure, key personnel, PM methodology, etc.)
- Evaluation criteria and weights for management section
- Specific staffing or qualification requirements
- Past performance requirements (number of references, relevance criteria)
**Validation**: All management-related evaluation criteria identified and mapped to response sections

### Step 2: Draft Organizational Overview
**Action**: Write the organizational qualifications section
**Input**: User-provided company information, past performance data
**Output**: Organizational overview including:
- Company overview and relevant capabilities
- Corporate experience in the domain
- Relevant certifications and accreditations
- Organizational resources and infrastructure
**Validation**: Claims about the organization are based on user-provided information (see `common/overconfidence-prevention.md`)

### Step 3: Draft Team Structure and Key Personnel
**Action**: Write the project team section
**Input**: User-provided team information, RFP staffing requirements
**Output**: Team structure section including:
- Organizational chart (described in text)
- Key personnel profiles (name, role, qualifications, relevant experience)
- Roles and responsibilities matrix
- Staffing plan (full-time, part-time, surge capacity)
- Staff retention and succession strategy
**Validation**: All RFP-required positions are filled; qualifications match requirements; key personnel exist (not fabricated)

### Step 4: Draft Project Management Methodology
**Action**: Write the project management approach section
**Input**: Content plan, technical response (for alignment), user methodology details
**Output**: PM methodology section including:
- Project management framework (PMI, PRINCE2, Agile PM, etc.)
- Governance structure and decision-making processes
- Communication plan (meetings, reports, escalation)
- Change management process
- Performance monitoring and reporting
**Validation**: PM methodology is compatible with the technical approach; reporting frequency meets RFP requirements

### Step 5: Draft Past Performance / References
**Action**: Write the past performance section
**Input**: User-provided past performance data
**Output**: Past performance section including:
- Relevant contract summaries (scope, value, duration)
- Performance outcomes and metrics
- Relevance to current RFP requirements
- Reference contact information
**Validation**: Past performance examples meet RFP's relevance criteria; contact information is current (confirmed by user)

### Step 6: Assemble and Validate Management Response
**Action**: Combine all sections into a cohesive management response
**Input**: All drafted sections from Steps 2-5
**Output**: Complete management response in `proposal-docs/proposal/management-response.md`
**Validation**:
- Total length within page budget
- Team structure aligns with technical approach staffing references
- Past performance examples are relevant to RFP requirements
- Content is consistent with executive summary and technical response
- Content validated per content-validation.md

---

## Question Generation

### Question Categories
- **Team Information**: Questions about key personnel names, qualifications, and availability
- **Corporate Experience**: Questions about relevant past contracts and performance
- **PM Methodology**: Questions about organizational project management practices
- **References**: Questions about available references and their contact status

### Question File Template

```markdown
# Management Response Questions

Please answer the following questions about your organization and team.

## Question 1
Who will serve as the Project Manager for this engagement?

A) [Name if previously mentioned] — Confirmed and available
B) To be determined — We'll assign from our PM pool
C) We need to hire for this role
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### Management Response
- **File**: `proposal-docs/proposal/management-response.md`
- **Content**: Complete management/organizational proposal section
- **Format**: Following the proposal section pattern from `common/output-structure-patterns.md`

---

## Completion Message

### REVIEW REQUIRED

**Management Response is complete.** Here's what was produced:

- Organizational overview with qualifications
- Team structure with [N] key personnel profiles
- Project management methodology: [framework name]
- [N] past performance examples with references
- [N]/[N] management requirements addressed

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the management response based on your feedback
**B) Continue to Pricing Framework** — Proceed to the pricing section (or next applicable stage)

---

## Error Handling

**Error**: Key personnel information not provided by user
- **Solution**: Generate question file requesting specific personnel details
- **Workaround**: Use role-based descriptions with "Key Personnel to be named upon award" for non-critical roles
- **Critical**: Named key personnel requirements in the RFP MUST have real names — do not fabricate

**Error**: Past performance examples don't meet RFP relevance criteria
- **Solution**: Ask user for additional examples; assess partial relevance
- **Workaround**: Present available examples with relevance rationale explaining how they relate

**Error**: Team structure doesn't align with technical approach staffing
- **Solution**: Cross-reference with technical response; reconcile differences
- **Recovery**: Update either team structure or technical approach staffing to be consistent

See `common/error-handling.md` for general error handling procedures.
