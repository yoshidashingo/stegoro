# Win Strategy — Planning Phase

## Purpose

This stage develops the competitive differentiation strategy for the proposal. Win themes, discriminators, and ghost strategies are defined here and then integrated into every subsequent proposal section during the Writing phase.

## Prerequisites
- Analysis phase completed (all three stages)
- Proposal Structure Design completed
- `proposal-docs/analysis/rfp-summary.md` — Competitive context assessment
- `proposal-docs/analysis/requirements.md` — Evaluation criteria and weights
- `proposal-docs/analysis/compliance-matrix.md` — Compliance strengths and gaps
- `proposal-docs/planning/proposal-structure.md` — Section structure for theme placement
- Common rules loaded: `common/question-format-guide.md`, `common/overconfidence-prevention.md`

## Execution Classification
**Type**: CONDITIONAL

**Execute IF**:
- Competitive bid (multiple vendors expected)
- Evaluation criteria include qualitative scoring (not purely price-based)
- User requests differentiation strategy
- RFP includes oral presentation or demonstration phase

**Skip IF**:
- Sole source procurement
- User explicitly declines strategy development
- RFP evaluation is purely Lowest Price Technically Acceptable (LPTA)

---

## Execution Steps

### Step 1: Competitive Landscape Assessment
**Action**: Gather intelligence about the competitive environment
**Input**: RFP summary (competitive context), user knowledge
**Output**: Competitive landscape including:
- Known or likely competitors
- Incumbent contractor (if re-compete)
- Each competitor's likely strengths and weaknesses
- Industry positioning and market dynamics
**Validation**: Assessment is based on user-provided information, not assumptions (see `common/overconfidence-prevention.md`)

### Step 2: Win Theme Development
**Action**: Define 2-4 key win themes that will differentiate the proposal
**Input**: Competitive landscape, evaluation criteria, compliance matrix (strengths)
**Output**: For each win theme:
- Theme statement (one sentence, memorable, evaluator-focused)
- Supporting evidence (proof points, past performance, metrics)
- Evaluation criteria alignment (which criteria this theme addresses)
- Competitive advantage (why this differentiates from competitors)
**Validation**: Each theme is specific and verifiable, not generic; themes collectively cover highest-weight evaluation criteria

### Step 3: Discriminator Identification
**Action**: Identify specific, verifiable discriminators
**Input**: Win themes, organizational capabilities, competitive assessment
**Output**: For each discriminator:
- Discriminator statement (specific capability or offering)
- Proof point (evidence that supports the claim)
- Competitive comparison (how this differs from competitors' likely offerings)
- Customer benefit (how the evaluator benefits)
**Validation**: Discriminators are specific (not generic strengths), verifiable, and relevant to evaluation criteria

### Step 4: Ghost Strategy Development
**Action**: Develop strategies to indirectly address competitor weaknesses
**Input**: Competitive landscape (competitor weaknesses), win themes
**Output**: For each ghost strategy:
- Competitor weakness being addressed
- Proposal language that highlights your strength in this area (without naming competitors)
- Section placement in the proposal
**Validation**: Ghost strategies are professional and indirect (never name competitors or disparage)

### Step 5: Theme Integration Plan
**Action**: Map win themes to proposal sections for consistent integration
**Input**: Win themes, discriminators, proposal structure
**Output**: Theme integration matrix:
- Which themes appear in which sections
- How each theme is expressed in each section (different emphasis per section)
- Key phrases and talking points for each section
- Graphics or visuals that reinforce themes
**Validation**: Each theme appears in at least 3 proposal sections; executive summary includes all themes

### Step 6: Generate Win Strategy Document
**Action**: Create the complete win strategy document
**Input**: All outputs from Steps 1-5
**Output**: `proposal-docs/planning/win-strategy.md`
**Validation**: Strategy is complete, internally consistent, and actionable for the Writing phase

---

## Question Generation

### Question Categories
- **Competitive Intelligence**: Questions about known competitors and their strengths/weaknesses
- **Organizational Strengths**: Questions about unique capabilities and differentiators
- **Past Performance**: Questions about relevant past projects and outcomes
- **Strategic Preferences**: Questions about positioning and messaging priorities

### Question File Template

```markdown
# Win Strategy Questions

Please answer the following questions to help develop the competitive strategy.

## Question 1
Who are the most likely competitors for this RFP?

A) [Company A] — We've competed against them before
B) [Company B] — They have incumbent advantage
C) Unknown — We're not sure who will bid
D) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 2
What is your organization's strongest differentiator for this opportunity?

A) Technical expertise — Deep domain knowledge and innovative approach
B) Past performance — Proven track record on similar contracts
C) Cost efficiency — Competitive pricing with strong value proposition
D) Team — Exceptional key personnel and team composition
E) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### Win Strategy Document
- **File**: `proposal-docs/planning/win-strategy.md`
- **Content**: Complete win strategy with themes, discriminators, ghost strategies, and integration plan
- **Format**:

```markdown
# Win Strategy — [RFP Title]

## Competitive Landscape
[Summary of competitive environment]

## Win Themes
### Theme 1: [Theme Statement]
- **Evidence**: [Proof points]
- **Criteria Alignment**: [Evaluation criteria served]
- **Competitive Advantage**: [How this differentiates]

[Repeat for each theme]

## Discriminators
| Discriminator | Proof Point | Customer Benefit |
|--------------|-------------|------------------|
| [Specific capability] | [Evidence] | [Benefit to customer] |

## Theme Integration Plan
| Proposal Section | Theme 1 | Theme 2 | Theme 3 |
|-----------------|---------|---------|---------|
| Executive Summary | [Key message] | [Key message] | [Key message] |
| Technical Approach | [Key message] | [Key message] | — |
| ... | ... | ... | ... |
```

---

## Completion Message

### REVIEW REQUIRED

**Win Strategy is complete.** Here's what was produced:

- [N] win themes developed with supporting evidence
- [N] discriminators identified
- Ghost strategy for [N] competitive scenarios
- Theme integration plan mapped across all proposal sections

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the win strategy based on your feedback
**B) Continue to Content Planning** — Proceed to detailed content planning

---

## Error Handling

**Error**: No competitive intelligence available
- **Solution**: Focus on strengths-based differentiation; skip ghost strategy
- **Workaround**: Develop themes based on evaluation criteria alignment and organizational strengths

**Error**: Win themes overlap or conflict with each other
- **Solution**: Consolidate overlapping themes; ensure each is distinct and independently valuable
- **Recovery**: Reduce theme count from 4 to 2-3 if needed for clarity

**Error**: Discriminators cannot be verified with available evidence
- **Solution**: Ask user for specific proof points, metrics, or case studies
- **Do Not Proceed**: With unsubstantiated discriminators (see `common/overconfidence-prevention.md`)

See `common/error-handling.md` for general error handling procedures.
