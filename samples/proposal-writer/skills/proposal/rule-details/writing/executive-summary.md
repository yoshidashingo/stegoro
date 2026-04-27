# Executive Summary — Writing Phase

## Purpose

This stage creates the executive summary — the single most important section of the proposal. The executive summary is often read by senior decision-makers who may not review the full technical response, making it critical for establishing a favorable first impression.

## Prerequisites
- Analysis phase completed
- Planning phase completed
- All prior artifacts loaded:
  - `proposal-docs/analysis/rfp-summary.md`
  - `proposal-docs/analysis/compliance-matrix.md`
  - `proposal-docs/planning/proposal-structure.md`
  - `proposal-docs/planning/win-strategy.md` (if exists)
  - `proposal-docs/planning/content-plan.md`
- Common rules loaded: `common/content-validation.md`, `common/overconfidence-prevention.md`

## Execution Classification
**Type**: ALWAYS
The executive summary always executes because every proposal requires a high-level summary for decision-makers.

## Adaptive Depth
- **Minimal**: Simple RFP — 1 page summary covering scope, approach, and qualifications
- **Standard**: Moderate RFP — 2-3 pages with structured sections, win themes, and evidence
- **Comprehensive**: Complex RFP — 3-5 pages with detailed solution overview, differentiators, visuals, and call to action

---

## Execution Steps

### Step 1: Executive Summary Outline (Part 1 — Planning)
**Action**: Create the executive summary outline for user approval
**Input**: Content plan, win strategy (if available), RFP summary
**Output**: Outline with:
- Opening approach (customer mission alignment)
- Problem/need restatement
- Solution overview structure
- Win themes placement
- Key differentiators to highlight
- Closing approach (call to action)
**Validation**: Outline addresses the highest-weight evaluation criteria; all win themes are included (if win strategy exists)

### Step 2: User Approval of Outline
**Action**: Present outline for user feedback
**Input**: Outline from Step 1
**Output**: Approved outline (or revised based on feedback)
**Validation**: User has explicitly approved the outline before drafting begins

### Step 3: Draft Opening Section (Part 2 — Generation)
**Action**: Write the opening paragraph(s) aligned with the customer's mission
**Input**: RFP summary (issuing organization, scope), win themes
**Output**: Opening that:
- Demonstrates understanding of the customer's mission and needs
- References specific language from the RFP
- Establishes credibility immediately
- Flows naturally into the problem/need discussion
**Validation**: Opening uses customer-centric language (not self-focused); references actual RFP content

### Step 4: Draft Problem/Need Summary
**Action**: Restate the problem or need being addressed
**Input**: RFP scope, requirements summary
**Output**: Problem summary that:
- Confirms understanding of what is being procured
- Highlights key challenges the customer faces
- Bridges to the solution overview
**Validation**: Problem statement aligns with RFP's stated objectives; no misinterpretation of scope

### Step 5: Draft Solution Overview
**Action**: Provide a high-level description of the proposed solution
**Input**: Content plan (technical approach summary), win themes
**Output**: Solution overview that:
- Describes the overall approach (without excessive technical detail)
- Highlights unique aspects of the solution
- Maps solution elements to customer needs
- Integrates win themes naturally
**Validation**: Solution overview is understandable by non-technical decision-makers; key proposal sections are foreshadowed

### Step 6: Draft Differentiators and Value Proposition
**Action**: Articulate why this proposal should be selected
**Input**: Win strategy (discriminators), past performance highlights
**Output**: Differentiators section that:
- Presents 2-4 key discriminators with supporting evidence
- Quantifies value where possible (metrics, savings, performance improvements)
- References relevant past performance
**Validation**: Every discriminator has a proof point; claims are verifiable (see `common/overconfidence-prevention.md`)

### Step 7: Draft Closing Statement
**Action**: Write a compelling closing that reinforces confidence
**Input**: Win themes, solution overview summary
**Output**: Closing that:
- Summarizes the key value proposition in 1-2 sentences
- Expresses commitment to the customer's success
- Includes forward-looking statement (implementation readiness)
**Validation**: Closing is confident without being arrogant; aligns with overall proposal tone

### Step 8: Assemble and Validate Executive Summary
**Action**: Combine all sections into a cohesive executive summary
**Input**: All drafted sections from Steps 3-7
**Output**: Complete executive summary in `proposal-docs/proposal/executive-summary.md`
**Validation**:
- Total length within page budget from proposal structure
- Win themes appear consistently
- Cross-references to other proposal sections are accurate
- No unverified claims (per overconfidence-prevention.md)
- Content validated per content-validation.md

---

## Output Artifacts

### Executive Summary
- **File**: `proposal-docs/proposal/executive-summary.md`
- **Content**: Complete executive summary ready for review
- **Format**: Following the proposal section pattern from `common/output-structure-patterns.md`

---

## Completion Message

### REVIEW REQUIRED

**Executive Summary is complete.** Here's what was produced:

- [N]-page executive summary
- Opening aligned with [customer name]'s mission
- Solution overview with [N] key differentiators highlighted
- Win themes integrated: [Theme 1], [Theme 2], [Theme 3]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the executive summary based on your feedback
**B) Continue to Technical Response** — Proceed to the technical proposal section

---

## Error Handling

**Error**: Executive summary exceeds page budget
- **Solution**: Identify lowest-priority content for trimming; present options to user
- **Recovery**: Condense subsections proportionally or move detail to technical response

**Error**: Win themes feel forced or disconnected from content flow
- **Solution**: Rephrase theme integration for more natural flow; reduce explicit theme callouts
- **Recovery**: Integrate themes through evidence and examples rather than direct statements

**Error**: User-provided evidence is insufficient for claimed discriminators
- **Solution**: Ask user for additional proof points or remove the claim
- **Do Not Include**: Unsubstantiated claims in the executive summary (see `common/overconfidence-prevention.md`)

See `common/error-handling.md` for general error handling procedures.
