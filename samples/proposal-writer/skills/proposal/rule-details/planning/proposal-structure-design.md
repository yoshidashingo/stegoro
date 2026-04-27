# Proposal Structure Design — Planning Phase

## Purpose

This stage designs the table of contents, volume structure, and section organization for the proposal. The structure ensures every RFP requirement is addressed within the proposal's format constraints while maximizing evaluation scoring potential.

## Prerequisites
- Analysis phase completed (all three stages)
- `proposal-docs/analysis/rfp-summary.md` — RFP metadata and structure
- `proposal-docs/analysis/requirements.md` — Categorized requirements
- `proposal-docs/analysis/compliance-matrix.md` — Compliance status and section mapping
- Common rules loaded: `common/content-validation.md`, `common/output-structure-patterns.md`

## Execution Classification
**Type**: ALWAYS
Proposal structure design always executes because every proposal needs an organized response structure.

## Adaptive Depth
- **Minimal**: Simple RFP that prescribes the exact format — Mirror the RFP's structure directly
- **Standard**: RFP provides general guidance with flexibility — Design optimized structure within guidelines
- **Comprehensive**: Complex, multi-volume RFP with significant structure flexibility — Design detailed structure with cross-volume references and strategic section ordering

---

## Execution Steps

### Step 1: Assess Structure Constraints
**Action**: Identify all structural constraints from the RFP
**Input**: RFP summary (submission requirements section)
**Output**: Structure constraints including:
- Prescribed section order (if RFP mandates specific sections)
- Volume structure (single vs. multi-volume)
- Page limits (overall and per section)
- Format requirements (headers, footers, fonts, margins)
- Required attachments and forms
**Validation**: All RFP structural requirements captured; no conflicting constraints

### Step 2: Determine Response Components
**Action**: Determine which proposal components are needed
**Input**: Requirements document, compliance matrix, RFP evaluation criteria
**Output**: Component list:
- Executive Summary (ALWAYS)
- Technical Response / Technical Approach (ALWAYS)
- Management Response / Management Approach (if required by RFP)
- Past Performance Volume (if required by RFP)
- Pricing / Cost Proposal (if required by RFP)
- Appendices (certifications, resumes, forms, etc.)
**Validation**: All RFP-required components identified; no unnecessary components included

### Step 3: Design Section Hierarchy
**Action**: Create the detailed table of contents
**Input**: Component list, RFP requirements, evaluation criteria weights
**Output**: Complete section hierarchy with:
- Section numbers and titles
- Brief description of each section's purpose
- RFP requirements addressed by each section
- Estimated page allocation
**Validation**: Every mandatory requirement maps to at least one section; page allocations sum to within page limit

### Step 4: Page Budget Allocation
**Action**: Allocate pages to sections based on evaluation weight and content needs
**Input**: Section hierarchy, page limits, evaluation criteria weights
**Output**: Page budget:
- Pages allocated to each section
- Buffer pages (typically 5-10% of total)
- Priority ranking for expansion/contraction if limits are tight
**Validation**: Total pages within limit; highest-weight sections receive proportionally more pages

### Step 5: Cross-Reference Strategy
**Action**: Plan how sections will reference each other to avoid redundancy
**Input**: Section hierarchy, requirements-to-section mapping
**Output**: Cross-reference plan:
- Requirements addressed in multiple sections (primary vs. secondary treatment)
- Forward references (Section X will detail this in Section Y)
- Back references (As described in Section X, ...)
- Compliance matrix cross-references
**Validation**: No requirement addressed only by cross-reference (must have at least one primary treatment)

### Step 6: Generate Proposal Structure Document
**Action**: Create the final proposal structure document
**Input**: All outputs from Steps 1-5
**Output**: `proposal-docs/planning/proposal-structure.md`
**Validation**: Structure is complete, internally consistent, and respects all RFP constraints

---

## Output Artifacts

### Proposal Structure Document
- **File**: `proposal-docs/planning/proposal-structure.md`
- **Content**: Complete table of contents with section descriptions, page budgets, and requirement mapping
- **Format**:

```markdown
# Proposal Structure — [RFP Title]

## Volume Structure
[Single volume / Multi-volume description]

## Table of Contents

### Volume 1: Technical Proposal
| Section | Title | Pages | Key Requirements | Eval Criteria |
|---------|-------|-------|-----------------|---------------|
| 1.0 | Executive Summary | [N] | Overview | All |
| 2.0 | Technical Approach | [N] | [Req IDs] | Technical Merit |
| ... | ... | ... | ... | ... |

### Page Budget Summary
| Component | Allocated | Limit | Status |
|-----------|-----------|-------|--------|
| Technical Volume | [N] | [N] | Within limit |
| ... | ... | ... | ... |

## Cross-Reference Map
[Details of how requirements are addressed across sections]
```

---

## Completion Message

### REVIEW REQUIRED

**Proposal Structure Design is complete.** Here's what was produced:

- [N]-section proposal structure designed
- Page budget allocated ([N] pages total, limit: [N])
- All [N] mandatory requirements mapped to proposal sections
- Cross-reference strategy defined

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the proposal structure based on your feedback
**B) Continue to Win Strategy** — Proceed to competitive differentiation (or Content Planning if Win Strategy is skipped)

---

## Error Handling

**Error**: Page limit is insufficient for all required content
- **Solution**: Present trade-off options: prioritize high-weight sections, use appendices for supplementary detail, condense lower-priority sections
- **Escalation**: Ask user to decide priorities

**Error**: RFP's prescribed format conflicts with optimal proposal flow
- **Solution**: Follow the RFP's format exactly (compliance over optimization), note flow issues for user awareness
- **Do Not Deviate**: Always follow RFP-prescribed structure when mandated

**Error**: Requirements map to no logical proposal section
- **Solution**: Create a dedicated section or subsection for orphan requirements
- **Workaround**: Include in the most closely related section with explicit callout

See `common/error-handling.md` for general error handling procedures.
