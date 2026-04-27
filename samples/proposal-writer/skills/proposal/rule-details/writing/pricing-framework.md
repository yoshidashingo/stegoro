# Pricing Framework — Writing Phase

## Purpose

This stage creates the pricing/cost proposal framework, including cost structure, rate tables, pricing methodology, and cost narrative. The pricing framework provides the structure and rationale — actual pricing numbers must be provided and validated by the user.

## Prerequisites
- Analysis and Planning phases completed
- Executive Summary and Technical Response completed
- Management Response completed (if executed)
- All prior artifacts loaded:
  - `proposal-docs/analysis/requirements.md` (pricing-related requirements)
  - `proposal-docs/analysis/compliance-matrix.md` (pricing compliance status)
  - `proposal-docs/planning/proposal-structure.md` (pricing section structure)
  - `proposal-docs/proposal/technical-response.md` (for alignment with technical scope)
  - `proposal-docs/proposal/management-response.md` (for staffing alignment, if exists)
- Common rules loaded: `common/content-validation.md`, `common/overconfidence-prevention.md`

## Execution Classification
**Type**: CONDITIONAL

**Execute IF**:
- RFP requests cost/pricing proposal
- Evaluation criteria include price scoring or cost realism analysis
- Budget or cost structure section required
- Contract type requires detailed pricing (FFP, T&M, CPFF, etc.)

**Skip IF**:
- Pricing handled separately outside this workflow
- RFP does not request pricing information
- User explicitly skips pricing section

**IMPORTANT**: This stage creates the pricing FRAMEWORK and NARRATIVE. It does NOT generate actual prices, rates, or dollar amounts. All financial figures must be provided and approved by the user.

---

## Execution Steps

### Step 1: Pricing Requirements Analysis
**Action**: Identify all pricing requirements and constraints from the RFP
**Input**: RFP (pricing instructions), requirements document
**Output**: Pricing requirements including:
- Contract type (FFP, T&M, CPFF, etc.)
- Required pricing format and templates
- Labor category definitions required
- Cost breakdown structure requirements
- Period of performance for pricing
- Option years and pricing escalation requirements
- Evaluation methodology for price (LPTA, best value, cost realism)
**Validation**: All pricing instructions from RFP captured; format requirements clear

### Step 2: Cost Breakdown Structure Design
**Action**: Design the cost breakdown structure aligned with technical approach
**Input**: Pricing requirements, technical response (scope and phases)
**Output**: Cost breakdown structure including:
- Major cost categories (labor, materials, travel, ODCs, etc.)
- Labor categories aligned with team structure
- Task/phase breakdown aligned with implementation plan
- CLIN structure (if required by RFP)
**Validation**: Cost structure maps completely to technical scope; no orphan costs or unpriced scope elements

### Step 3: Rate Table Framework
**Action**: Create the framework for rate tables and labor categories
**Input**: RFP labor category requirements, team structure
**Output**: Rate table framework including:
- Labor categories with descriptions
- Skill level definitions per category
- Rate table structure (base year, option years)
- Escalation methodology (if multi-year)
**Validation**: All RFP-required labor categories included; descriptions align with personnel qualifications from management response

### Step 4: Pricing Methodology Documentation
**Action**: Write the pricing methodology description
**Input**: Contract type, pricing requirements, organizational pricing approach
**Output**: Methodology section including:
- Basis of estimate (how prices were developed)
- Direct cost estimation methodology
- Indirect rate application (if applicable)
- Profit/fee calculation approach
- Assumptions and exclusions
**Validation**: Methodology is transparent and defensible; assumptions are clearly stated

### Step 5: Cost Narrative
**Action**: Write the narrative explaining the pricing rationale
**Input**: Pricing methodology, technical approach, win strategy (value proposition)
**Output**: Cost narrative including:
- Summary of total proposed price (placeholder for user input)
- Explanation of pricing approach and value
- Cost efficiency measures and savings
- Cost risk discussion
- Value proposition linking price to quality of solution
**Validation**: Narrative supports the pricing without contradicting technical or management volumes

### Step 6: Assemble Pricing Framework
**Action**: Combine all sections into a cohesive pricing document
**Input**: All outputs from Steps 1-5
**Output**: Complete pricing framework in `proposal-docs/proposal/pricing-framework.md`
**Validation**:
- All RFP pricing requirements addressed
- Framework aligns with technical scope and staffing plan
- Placeholder locations clearly marked for user's actual pricing data
- Content validated per content-validation.md

---

## Question Generation

### Question Categories
- **Pricing Strategy**: Questions about pricing approach (aggressive, competitive, premium)
- **Rate Information**: Questions about labor rates and cost structure
- **Cost Constraints**: Questions about budget targets or price-to-win figures
- **Assumptions**: Questions about pricing assumptions (travel, materials, escalation)

### Question File Template

```markdown
# Pricing Framework Questions

Please answer the following questions about your pricing approach.

## Question 1
What pricing strategy should we target for this proposal?

A) Competitive — Price to win, prioritize affordability
B) Best value — Balance price with quality of solution
C) Premium — Price reflects superior solution and team
D) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 2
What contract type does the RFP specify?

A) Firm Fixed Price (FFP) — Fixed total price for defined deliverables
B) Time and Materials (T&M) — Hourly rates with materials at cost
C) Cost Plus Fixed Fee (CPFF) — Reimbursable costs plus negotiated fee
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Output Artifacts

### Pricing Framework
- **File**: `proposal-docs/proposal/pricing-framework.md`
- **Content**: Complete pricing structure, rate table framework, methodology, and narrative with placeholders for actual pricing data
- **Format**:

```markdown
# Pricing Framework — [RFP Title]

## Pricing Summary
- **Contract Type**: [Type]
- **Period of Performance**: [Duration]
- **Total Proposed Price**: [USER INPUT REQUIRED]

## Cost Breakdown Structure
| Category | Description | Amount |
|----------|-------------|--------|
| [Category] | [Description] | [USER INPUT] |

## Labor Rate Table
| Labor Category | Description | Base Year Rate | Option Year 1 |
|---------------|-------------|----------------|----------------|
| [Category] | [Description] | [USER INPUT] | [USER INPUT] |

## Pricing Methodology
[Description of how prices were developed]

## Cost Narrative
[Rationale and value proposition for the pricing]

## Assumptions and Exclusions
[List of pricing assumptions]
```

---

## Completion Message

### REVIEW REQUIRED

**Pricing Framework is complete.** Here's what was produced:

- Cost breakdown structure with [N] categories
- Rate table framework with [N] labor categories
- Pricing methodology documented
- Cost narrative with value proposition
- [N] placeholder fields requiring your pricing data

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the pricing framework based on your feedback
**B) Continue to Compliance Check** — Proceed to the Review phase

---

## Error Handling

**Error**: RFP pricing format requirements are highly specific or use proprietary templates
- **Solution**: Create framework in closest approximation; note where user must transfer to official templates
- **Workaround**: Provide content in structured format for manual transfer to required templates

**Error**: Cost breakdown structure doesn't align with technical scope
- **Solution**: Cross-reference with technical response implementation phases; reconcile differences
- **Recovery**: Update cost structure or flag scope elements that need pricing alignment

**Error**: User cannot provide rates or pricing data
- **Solution**: Create complete framework with all placeholders clearly marked; user fills in later
- **Note**: The framework is still valuable even without specific numbers

See `common/error-handling.md` for general error handling procedures.
