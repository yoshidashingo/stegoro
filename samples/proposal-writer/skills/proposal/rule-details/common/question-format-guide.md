# Question Format Guide

## Purpose

This file defines the standardized format for ALL questions asked during the proposal creation workflow. Consistent question formatting ensures clear communication, enables answer validation, and supports contradiction detection.

---

## Core Rules

### Rule 1: Questions Must Be in Dedicated Files
- NEVER ask questions directly in chat
- ALL questions must be generated as structured question files
- Question files are saved to the appropriate phase directory under `proposal-docs/`

### Rule 2: Multiple Choice Format
- Every question MUST use multiple choice format (A, B, C, D options)
- The LAST option MUST always be "Other (please describe after [Answer]: tag below)"
- Minimum 2 meaningful options + "Other" option
- Maximum 4 meaningful options + "Other" option

### Rule 3: [Answer]: Tag
- Every question MUST include an `[Answer]:` tag after the options
- Users respond by filling in the [Answer]: tag
- Responses can be letter codes (A, B, C) or free text after "Other"

### Rule 4: Answer Validation
- After receiving answers, validate for completeness and consistency
- If contradictions detected, generate a clarification question file
- Do not proceed with contradictory answers unresolved

---

## Question File Template

```markdown
# [Phase/Stage Name] Questions

Please answer the following questions to help [context-specific purpose].

## Question 1
[Clear, specific question text]

A) [First meaningful option]
B) [Second meaningful option]
C) [Third meaningful option]
D) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 2
[Clear, specific question text]

A) [First meaningful option]
B) [Second meaningful option]
C) [Third meaningful option]
D) [Fourth meaningful option]
E) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Clarification Question File Template

When contradictions or ambiguities are detected in user answers:

```markdown
# [Phase/Stage Name] Clarification Questions

I detected [contradictions/ambiguities] in your responses that need clarification:

## Issue 1: [Brief Description]
You indicated "[Answer A]" (Q[X]:[Letter]) but also "[Answer B]" (Q[Y]:[Letter]).
[Explanation of why these are contradictory/ambiguous]

### Clarification Question 1
[Specific question to resolve the issue]

A) [Option resolving toward first answer]
B) [Option resolving toward second answer]
C) [Option providing middle ground]
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

---

## Phase-Specific Question Categories

### Analysis Phase Questions
- RFP scope and type clarification
- Requirement interpretation when ambiguous
- Priority between conflicting requirements
- Compliance capability assessment

### Planning Phase Questions
- Competitive landscape and positioning
- Organizational strengths and differentiators
- Content priorities and emphasis areas
- Team structure and key personnel availability

### Writing Phase Questions
- Technical approach preferences
- Past performance examples to reference
- Pricing strategy and constraints
- Section emphasis and detail level

### Review Phase Questions
- Compliance gap resolution strategies
- Quality trade-offs when page limits are tight
- Final formatting preferences
- Submission logistics

---

## Answer Processing Rules

### Step 1: Collect Answers
- Read user's completed question file
- Extract answers using [Answer]: tags
- Map letter responses to option text

### Step 2: Validate Completeness
- Check all questions have answers
- Identify any blank or unclear answers
- Flag unanswered questions for follow-up

### Step 3: Detect Contradictions
Common contradiction patterns in proposal context:
- Claiming "aggressive pricing" but also "premium service levels"
- Selecting "minimal team" but also "comprehensive project management"
- Choosing "rapid delivery" but also "extensive quality assurance"
- Indicating "fixed price" but also "scope flexibility"

### Step 4: Resolve or Proceed
- If contradictions found: Generate clarification question file
- If all clear: Log answers in audit.md and proceed

---

## Implementation Guidelines

1. **Be specific**: Questions should be precise enough that any answer drives a clear action
2. **Be relevant**: Only ask questions necessary for the current stage
3. **Provide context**: Each question should explain why the answer matters
4. **Limit questions**: No more than 7 questions per question file
5. **Order logically**: Questions should flow from general to specific

## Error Handling

- If user provides free text instead of letter codes: Parse intent and confirm interpretation
- If user skips questions: Highlight unanswered items and explain why they're needed
- If user provides contradictory answers: Generate clarification question file immediately

## References

- `common/overconfidence-prevention.md` — When to ask vs. when to decide
- `common/content-validation.md` — Validating question file content
- `common/error-handling.md` — Handling incomplete or invalid answers
