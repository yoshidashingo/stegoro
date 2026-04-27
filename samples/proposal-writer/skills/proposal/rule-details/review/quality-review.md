# Quality Review — Review Phase

## Purpose

This stage evaluates the overall quality of the proposal content including writing clarity, persuasiveness, consistency, technical accuracy, and alignment with win strategy. The quality review goes beyond compliance to assess whether the proposal is compelling enough to score well.

## Prerequisites
- Compliance Check completed
- `proposal-docs/review/compliance-check-report.md` — Compliance findings (critical gaps should already be addressed)
- All written proposal sections
- Win strategy document (if executed): `proposal-docs/planning/win-strategy.md`
- Content plan: `proposal-docs/planning/content-plan.md`
- Common rules loaded: `common/content-validation.md`, `common/quality-standards.md`

## Execution Classification
**Type**: ALWAYS
Quality review always executes because compliance alone does not win proposals — quality differentiates competitive entries.

---

## Execution Steps

### Step 1: Writing Quality Assessment
**Action**: Review the writing quality across all proposal sections
**Input**: All written proposal sections
**Output**: Writing quality findings:
- Clarity: Is the writing clear and unambiguous?
- Conciseness: Is the writing free of unnecessary repetition and filler?
- Active voice: Does the proposal use active, confident language?
- Customer focus: Is the proposal written from the evaluator's perspective?
- Professional tone: Is the tone appropriate (confident but not arrogant)?
**Validation**: Findings are specific with section references and examples

### Step 2: Persuasiveness Assessment
**Action**: Evaluate how persuasive and compelling the proposal is
**Input**: All proposal sections, win strategy (if available)
**Output**: Persuasiveness findings:
- Value proposition clarity: Is it clear why this proposal should be selected?
- Evidence strength: Are claims supported by specific, verifiable evidence?
- Differentiator visibility: Are key discriminators prominent and memorable?
- Benefit articulation: Are customer benefits clearly stated (not just features)?
- Call to action: Does the proposal create urgency and confidence?
**Validation**: Assessment considers evaluator perspective (would this score well against criteria?)

### Step 3: Win Theme Integration Review
**Action**: Verify win themes are consistently and effectively integrated (if win strategy was executed)
**Input**: All proposal sections, win strategy document, theme integration plan
**Output**: Theme integration findings:
- Each theme's presence across planned sections
- Theme consistency (same message, different emphasis per section)
- Theme support (each mention backed by evidence)
- Natural integration vs. forced insertion
**Validation**: Themes are present where planned; integration feels natural; evidence supports each mention

### Step 4: Technical Accuracy Review
**Action**: Verify technical content is accurate, feasible, and internally consistent
**Input**: Technical response, implementation plan, risk mitigation
**Output**: Technical accuracy findings:
- Methodology described is real and applicable
- Architecture components are compatible
- Timeline is realistic given scope and resources
- Risk assessments are appropriate (not over/understated)
- Technology choices are current (not outdated)
**Validation**: No technical claims that could be challenged by evaluators; timeline and resource estimates are defensible

### Step 5: Evaluator Experience Assessment
**Action**: Assess the proposal from the evaluator's perspective
**Input**: All proposal sections, evaluation criteria, page limits
**Output**: Evaluator experience findings:
- Ease of scoring: Can evaluators easily find responses to their criteria?
- Requirements traceability: Are RFP requirements explicitly called out in responses?
- Visual appeal: Are headers, tables, and formatting aiding comprehension?
- Section flow: Does the proposal read logically from start to finish?
- Redundancy: Is there unnecessary repetition between sections?
**Validation**: An evaluator could efficiently score this proposal against their evaluation criteria

### Step 6: Generate Quality Review Report
**Action**: Create the quality review report
**Input**: All findings from Steps 1-5
**Output**: `proposal-docs/review/quality-review-report.md`
**Validation**: Report is actionable with specific improvement recommendations

---

## Output Artifacts

### Quality Review Report
- **File**: `proposal-docs/review/quality-review-report.md`
- **Content**: Comprehensive quality assessment with findings and recommendations
- **Format**:

```markdown
# Quality Review Report — [RFP Title]

## Summary
- **Overall Quality Rating**: [Excellent / Good / Adequate / Needs Improvement]
- **Writing Quality**: [Rating]
- **Persuasiveness**: [Rating]
- **Win Theme Integration**: [Rating]
- **Technical Accuracy**: [Rating]
- **Evaluator Experience**: [Rating]

## Strengths
- [Specific strength with section reference]
- [Specific strength with section reference]

## Areas for Improvement

### Critical (Should fix before submission)
| # | Section | Issue | Recommendation |
|---|---------|-------|---------------|
| 1 | [Section] | [Issue description] | [Specific recommendation] |

### Recommended (Improves competitiveness)
| # | Section | Issue | Recommendation |
|---|---------|-------|---------------|
| 1 | [Section] | [Issue description] | [Specific recommendation] |

## Detailed Assessment

### Writing Quality
[Detailed findings]

### Persuasiveness
[Detailed findings]

### Win Theme Integration
[Detailed findings]

### Technical Accuracy
[Detailed findings]

### Evaluator Experience
[Detailed findings]

## Overall Assessment
[Paragraph summarizing the proposal's competitive readiness]
```

---

## Completion Message

### REVIEW REQUIRED

**Quality Review is complete.** Here's a summary:

- Overall quality: [Rating]
- [N] strengths identified
- [N] critical improvements recommended
- [N] optional enhancements suggested
- Win theme integration: [Assessment]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll address the quality findings before final assembly
**B) Continue to Final Assembly** — Proceed to compile the final proposal package

---

## Error Handling

**Error**: Quality review reveals significant writing quality issues across the proposal
- **Solution**: Present the most impactful improvements; offer to revise the weakest sections
- **Recovery**: Focus on sections that affect highest-weight evaluation criteria

**Error**: Win themes are missing or poorly integrated
- **Solution**: Identify specific sections where themes should appear; draft revised language
- **Recovery**: Focus theme integration on executive summary and technical approach (highest-visibility sections)

**Error**: Technical content appears infeasible or outdated
- **Solution**: Flag specific concerns; ask user to validate technical claims
- **Do Not Submit**: Proposals with known technical inaccuracies

See `common/error-handling.md` for general error handling procedures.
