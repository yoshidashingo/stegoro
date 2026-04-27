# Overconfidence Prevention

## Purpose

This file enforces the "When in doubt, ask" philosophy throughout the proposal creation workflow. Overconfidence — making assumptions instead of asking — is the primary risk in AI-assisted proposal writing because incorrect assumptions lead to non-compliant or off-target proposals.

---

## Core Principle

**Never assume when you can verify.** In proposal writing, wrong assumptions have compounding effects:
- A misunderstood requirement leads to a non-compliant response
- A non-compliant response leads to lost points or disqualification
- A missed evaluation criterion leads to an unscored section
- An incorrect differentiator leads to a weak competitive position

---

## When to Ask vs. When to Decide

### ALWAYS Ask (Never Assume)

**RFP Interpretation**:
- Ambiguous language in requirements ("should" vs. "shall" vs. "may")
- Requirements that could be interpreted multiple ways
- Evaluation criteria with unclear scoring methodology
- Submission requirements with multiple possible interpretations

**Company Capabilities**:
- Technical capabilities the user's organization possesses
- Past performance examples and contract references
- Team member qualifications and availability
- Pricing rates, labor categories, and cost structures

**Strategic Decisions**:
- Competitive positioning and differentiators
- Teaming arrangements and subcontracting
- Pricing strategy (aggressive, competitive, premium)
- Go/no-go decisions on specific requirements

**Compliance Decisions**:
- Whether to take exception to a requirement
- How to address requirements that cannot be fully met
- Alternative approaches to mandatory requirements
- Regulatory interpretations

### Can Decide (Standard Practice)

**Document Structure**:
- Standard proposal formatting conventions
- Section ordering within established frameworks
- Heading hierarchy and numbering
- Cross-reference formatting

**Writing Conventions**:
- Professional tone and style
- Active voice preference
- Standard proposal phrasing (e.g., "[Company] understands that...")
- Grammar and punctuation rules

**Workflow Mechanics**:
- Which common rules to load
- Validation checklist execution
- Audit log formatting
- State file updates

---

## Red Flags — Stop and Ask

The following situations indicate overconfidence risk. STOP and generate questions:

### Analysis Phase Red Flags
- RFP uses inconsistent terminology for the same concept
- Requirements reference standards or regulations not included in the RFP
- Page limit seems insufficient for the required response
- Evaluation weights don't sum to 100%
- Mandatory requirements seem to conflict with each other

### Planning Phase Red Flags
- Multiple viable proposal structures with different trade-offs
- Win themes based on assumptions about competitors
- Content plan requires technical detail not yet provided by user
- Proposal structure deviates from RFP's suggested format

### Writing Phase Red Flags
- Making specific claims about user's company capabilities without confirmation
- Citing metrics, statistics, or case studies not provided by user
- Describing technical approaches without user validation
- Committing to timelines, staffing levels, or pricing without user input

### Review Phase Red Flags
- Marking requirements as "Compliant" without verified response content
- Accepting quality trade-offs without user approval
- Making final formatting decisions that affect content
- Assuming submission logistics are handled

---

## Question Categories by Phase

### Analysis Phase
- "The RFP uses both 'system' and 'platform' — do these refer to the same thing?"
- "Requirement 3.2 says 'should' while 3.5 says 'shall' for similar items — should we treat both as mandatory?"
- "The evaluation criteria mention 'innovation' but don't define how it's scored — should we ask for clarification from the issuer?"

### Planning Phase
- "Should the proposal follow the exact section numbering from the RFP, or can we restructure for better flow?"
- "Who are the expected competitors, and what are our key differentiators against them?"
- "Does your organization have past performance on similar contracts in the last 3 years?"

### Writing Phase
- "What specific methodology does your organization use for [technical area]?"
- "Can you provide a specific case study or past performance example for this requirement?"
- "What labor categories and rates should be used in the pricing section?"

### Review Phase
- "Requirement 4.7 is addressed indirectly in Section 3.2 — is this sufficient, or should we add an explicit response?"
- "The proposal is 2 pages over the limit — which sections should be trimmed?"

---

## Success Indicators

The overconfidence prevention system is working when:

1. **Questions are generated** at each phase — not just during Analysis
2. **Assumptions are documented** and explicitly flagged for user review
3. **Contradictions are caught** before they reach the final proposal
4. **User provides corrections** at review checkpoints (indicating questions were needed)
5. **Compliance matrix** has no gaps caused by assumed interpretations

The system is FAILING when:
1. Proposal sections contain claims not verified with the user
2. Technical approaches are described without user input
3. Compliance is assumed rather than verified
4. Competitive positioning relies on unverified assumptions

---

## Implementation Guidelines

1. **Err on the side of asking**: A redundant question costs less than an incorrect assumption
2. **Group questions logically**: Batch related questions into a single question file per stage
3. **Provide context**: Explain why each question matters for the proposal
4. **Offer good defaults**: When asking, suggest the most likely answer as option A
5. **Track assumptions**: Log any unavoidable assumptions in audit.md for later validation

## Error Handling

- If user declines to answer: Note the gap, use conservative defaults, flag in compliance review
- If user provides partial answers: Ask targeted follow-up questions
- If user contradicts previous answers: Generate clarification question file
- See `common/error-handling.md` for general procedures

## References

- `common/question-format-guide.md` — How to format questions
- `common/error-handling.md` — General error handling
- `common/quality-standards.md` — Quality dimensions for overconfidence assessment
