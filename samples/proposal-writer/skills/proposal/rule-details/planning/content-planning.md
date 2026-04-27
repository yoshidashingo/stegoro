# Content Planning — Planning Phase

## Purpose

This stage creates detailed content outlines for every proposal section, specifying key messages, evidence needs, graphics plans, and compliance mapping. Content planning bridges the gap between strategy (what to say) and execution (how to write it).

## Prerequisites
- Analysis phase completed (all three stages)
- Proposal Structure Design completed
- Win Strategy completed (if executed) or skipped
- All prior artifacts loaded:
  - `proposal-docs/analysis/requirements.md`
  - `proposal-docs/analysis/compliance-matrix.md`
  - `proposal-docs/planning/proposal-structure.md`
  - `proposal-docs/planning/win-strategy.md` (if exists)
- Common rules loaded: `common/content-validation.md`, `common/overconfidence-prevention.md`

## Execution Classification
**Type**: ALWAYS
Content planning always executes because writing without a plan leads to inconsistent, non-compliant proposals.

## Adaptive Depth
- **Minimal**: Simple RFP — Section-level outlines with key points
- **Standard**: Moderate RFP — Detailed subsection outlines with evidence mapping and graphics identification
- **Comprehensive**: Complex RFP — Full paragraph-level outlines with specific proof points, graphics mockups, and cross-section coordination

---

## Execution Steps

### Step 1: Section Outline Creation
**Action**: Create a detailed outline for each proposal section
**Input**: Proposal structure, requirements, compliance matrix, win strategy (if available)
**Output**: For each section:
- Section heading and subheadings
- Key message for each subsection (one sentence summarizing the main point)
- Requirements addressed in each subsection
- Estimated paragraphs / page allocation within section
**Validation**: Every section from the proposal structure has an outline; no requirements are unaddressed

### Step 2: Evidence and Proof Point Mapping
**Action**: Identify the evidence needed to support each section's claims
**Input**: Section outlines, win themes, discriminators
**Output**: For each section:
- Proof points needed (metrics, case studies, certifications, references)
- Evidence source (user-provided, publicly available, to be requested)
- Evidence status (available, needed, pending)
**Validation**: Key claims have identified evidence; gaps flagged for user input

### Step 3: Graphics and Visual Elements Planning
**Action**: Identify where graphics, tables, diagrams, or figures will enhance the proposal
**Input**: Section outlines, evaluation criteria, page budget
**Output**: For each planned graphic:
- Type (org chart, timeline, architecture diagram, process flow, comparison table)
- Purpose (what it communicates)
- Location in proposal (section and approximate position)
- Estimated space (fraction of a page)
**Validation**: Graphics support key messages; graphics don't exceed page budget allocations

### Step 4: Compliance Mapping Verification
**Action**: Verify that every requirement is explicitly planned for in a specific section
**Input**: Section outlines, compliance matrix
**Output**: Updated compliance matrix with:
- Specific subsection where each requirement will be addressed
- Primary treatment vs. cross-reference indication
- Compliance response approach (direct response, alternative approach, exception)
**Validation**: 100% of mandatory requirements have a planned location; no requirement is addressed only by cross-reference

### Step 5: Win Theme Integration Verification
**Action**: Verify win themes are woven into the content plan (if win strategy was executed)
**Input**: Section outlines, theme integration plan from win strategy
**Output**: Verification that:
- Each win theme appears in the planned content of its assigned sections
- Theme messaging is consistent but varied across sections (not repetitive)
- Discriminators are supported by planned evidence
**Validation**: Theme integration plan is achievable within the content outlines

### Step 6: Content Dependencies Identification
**Action**: Identify what information is still needed from the user before writing can begin
**Input**: Evidence mapping, section outlines
**Output**: Content dependencies list:
- Information needed from user (capabilities, past performance, team details, pricing data)
- Priority (blocking vs. nice-to-have)
- Default approach if information not available
**Validation**: All blocking dependencies identified; non-blocking items have fallback approaches

### Step 7: Generate Content Plan Document
**Action**: Create the comprehensive content plan
**Input**: All outputs from Steps 1-6
**Output**: `proposal-docs/planning/content-plan.md`
**Validation**: Plan is complete, actionable, and traceable to requirements and strategy

---

## Question Generation

### Question Categories
- **Evidence Availability**: Questions about specific proof points and case studies
- **Technical Details**: Questions about methodology, tools, and approach specifics
- **Team Information**: Questions about key personnel and organizational capabilities
- **Content Priorities**: Questions about emphasis areas when space is limited

### Question File Template

```markdown
# Content Planning Questions

Please answer the following questions to finalize the content plan before writing begins.

## Question 1
For the Technical Approach section, which methodology does your organization use?

A) Agile / Scrum — Iterative delivery with sprint-based planning
B) Waterfall — Sequential phases with stage gates
C) Hybrid — Combined approach tailored to project needs
D) Organization-specific methodology (please describe after [Answer]: tag)
E) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### Content Plan Document
- **File**: `proposal-docs/planning/content-plan.md`
- **Content**: Section-by-section content outline with evidence mapping, graphics plan, and compliance verification
- **Format**:

```markdown
# Content Plan — [RFP Title]

## Section Outlines

### Section 1: Executive Summary
**Page Budget**: [N] pages
**Key Message**: [One-sentence summary]
**Win Themes**: [Themes integrated]

#### Subsection Outlines
- 1.1 Opening / Customer Mission Alignment
  - Key Point: [message]
  - Evidence: [proof point]
- 1.2 Solution Overview
  - Key Point: [message]
  - Evidence: [proof point]
  - Graphic: [type — description]

[Continue for all sections]

## Evidence Status
| Evidence Item | Source | Status | Priority |
|--------------|--------|--------|----------|
| [Item] | [Source] | Available / Needed / Pending | High / Medium / Low |

## Graphics Plan
| # | Type | Section | Purpose | Size |
|---|------|---------|---------|------|
| 1 | [Type] | [Section] | [Purpose] | [Pages] |

## Content Dependencies
| Item | Priority | Fallback Approach |
|------|----------|------------------|
| [Information needed] | Blocking / Nice-to-have | [Default if not provided] |
```

---

## Completion Message

### REVIEW REQUIRED

**Content Planning is complete.** Here's what was produced:

- Detailed outlines for [N] proposal sections
- [N] evidence items identified ([N] available, [N] needed from you)
- [N] graphics/visuals planned
- 100% of mandatory requirements mapped to specific subsections
- [N] content dependencies requiring your input

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the content plan based on your feedback
**B) Continue to Executive Summary** — Proceed to the Writing phase

---

## Error Handling

**Error**: Content plan reveals sections without enough source material
- **Solution**: Generate targeted question file for missing information
- **Workaround**: Plan section with available information, mark gaps for user review during Writing phase

**Error**: Page budget is insufficient for all planned content
- **Solution**: Present priority-based content reduction options to user
- **Recovery**: Reduce lower-priority subsections; move supplementary detail to appendices

**Error**: Win themes cannot be naturally integrated into all planned sections
- **Solution**: Reduce theme placement to sections where integration is natural and relevant
- **Do Not Force**: Artificial theme integration that disrupts content flow

See `common/error-handling.md` for general error handling procedures.
