# Question Format Guide

## MANDATORY: All Questions Must Use This Format

### Rule: Never Ask Questions in Chat
**CRITICAL**: You must NEVER ask questions directly in the chat. ALL questions must be placed in dedicated question files.

### Question File Format

#### File Naming Convention
- Use descriptive names: `{phase-name}-questions.md`
- Place in the working artifact directory: `steering-docs/{phase-name}/`
- Examples:
  - `purpose-analysis-questions.md`
  - `domain-research-questions.md`
  - `workflow-architecture-questions.md`
  - `common-rules-design-questions.md`

#### Question Structure
Every question must include meaningful options plus "Other" as the last option:

```markdown
## Question [Number]
[Clear, specific question text]

A) [First meaningful option]
B) [Second meaningful option]
[...additional options as needed...]
X) Other (please describe after [Answer]: tag below)

[Answer]:
```

**CRITICAL**:
- "Other" is MANDATORY as the LAST option for every question
- Only include meaningful options — don't make up options to fill slots
- Use as many or as few options as make sense (minimum 2 + Other)

### Complete Example

```markdown
# Purpose Analysis Clarification Questions

Please answer the following questions to help clarify the target agent's purpose.

## Question 1
What is the primary function of the target AI agent?

A) Guiding users through a multi-step process (Process Agent)
B) Executing a specific bounded task (Task Agent)
C) Analyzing inputs and producing structured outputs (Analytical Agent)
D) Combining multiple operational modes (Hybrid Agent)
E) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 2
How many distinct user types will interact with the target agent?

A) Single user type with clear needs
B) 2-3 user types with related needs
C) 4+ user types with diverse needs
D) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 3
What is the expected complexity of the target agent's workflow?

A) Simple — 2-3 stages, linear flow
B) Standard — 4-6 stages, some branching
C) Complex — 7+ stages, conditional execution, state tracking
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

### User Response Format
Users will answer by filling in the letter choice after [Answer]: tag:

```markdown
## Question 1
What is the primary function of the target AI agent?

A) Guiding users through a multi-step process (Process Agent)
B) Executing a specific bounded task (Task Agent)
C) Analyzing inputs and producing structured outputs (Analytical Agent)
D) Combining multiple operational modes (Hybrid Agent)
E) Other (please describe after [Answer]: tag below)

[Answer]: A
```

### Reading User Responses
After user confirms completion:
1. Read the question file
2. Extract answers after [Answer]: tags
3. Validate all questions are answered
4. Proceed with analysis based on responses

### Multiple Choice Guidelines

#### Option Count
- Minimum: 2 meaningful options + "Other" (A, B, C)
- Typical: 3-4 meaningful options + "Other" (A, B, C, D, E)
- Maximum: 5 meaningful options + "Other" (A, B, C, D, E, F)
- **CRITICAL**: Don't make up options just to fill slots — only include meaningful choices

#### Option Quality
- Make options mutually exclusive
- Cover the most common scenarios
- Only include meaningful, realistic options
- **ALWAYS include "Other" as the LAST option** (MANDATORY)
- Be specific and clear
- **Don't make up options to fill A, B, C, D slots**

#### Good Example:
```markdown
## Question 5
What domain will the target agent operate in?

A) Software development (code, testing, deployment)
B) Content creation (documentation, writing, design)
C) Data processing (analysis, transformation, reporting)
D) Compliance and auditing (standards, regulations, quality)
E) Other (please describe after [Answer]: tag below)

[Answer]:
```

#### Bad Example (Avoid):
```markdown
## Question 5
What will the agent do?

A) Yes
B) No
C) Maybe

[Answer]:
```

### Workflow Integration

#### Step 1: Create Question File
```markdown
Create steering-docs/{phase-name}/{phase-name}-questions.md with all questions
```

#### Step 2: Inform User
```
"I've created {phase-name}-questions.md with [X] questions.
Please answer each question by filling in the letter choice after the [Answer]: tag.
If none of the options match your needs, choose the last option (Other) and describe your preference. Let me know when you're done."
```

#### Step 3: Wait for Confirmation
Wait for user to say "done", "completed", "finished", or similar.

#### Step 4: Read and Analyze
```
Read steering-docs/{phase-name}/{phase-name}-questions.md
Extract all answers
Validate completeness
Proceed with analysis
```

### Error Handling

#### Missing Answers
If any [Answer]: tag is empty:
```
"I noticed Question [X] is not answered. Please provide an answer using one of the letter choices
for all questions before proceeding."
```

#### Invalid Answers
If answer is not a valid letter choice:
```
"Question [X] has an invalid answer '[answer]'.
Please use only the letter choices provided in the question."
```

#### Ambiguous Answers
If user provides explanation instead of letter:
```
"For Question [X], please provide the letter choice that best matches your answer.
If none match, choose 'Other' and add your description after the [Answer]: tag."
```

### Contradiction and Ambiguity Detection

**MANDATORY**: After reading user responses, you MUST check for contradictions and ambiguities.

#### Detecting Contradictions
Look for logically inconsistent answers:
- Scope mismatch: "Simple task agent" but "7+ phases needed"
- Complexity mismatch: "Minimal depth" but "high-risk domain"
- Stakeholder mismatch: "Single user type" but "role-based access needed"
- Type mismatch: "Analytical agent" but "multi-phase sequential workflow"

#### Detecting Ambiguities
Look for unclear or borderline responses:
- Answers that could fit multiple classifications
- Responses that lack specificity
- Conflicting indicators across multiple questions

#### Creating Clarification Questions
If contradictions or ambiguities detected:

1. **Create clarification file**: `{phase-name}-clarification-questions.md`
2. **Explain the issue**: Clearly state what contradiction/ambiguity was detected
3. **Ask targeted questions**: Use multiple choice format to resolve the issue
4. **Reference original questions**: Show which questions had conflicting answers

**Example**:
```markdown
# Purpose Analysis Clarification Questions

I detected contradictions in your responses that need clarification:

## Contradiction 1: Agent Type vs Complexity
You indicated "Task Agent" (Q1:B) but also "7+ stages with conditional execution" (Q3:C).
Task agents typically have 2-4 stages. This suggests a more complex agent type.

### Clarification Question 1
Which better describes the target agent?

A) A focused task agent with a few well-defined stages (simpler)
B) A process agent that guides through multiple conditional stages (more complex)
C) A hybrid that sometimes acts as a task agent and sometimes as a process agent
D) Other (please describe after [Answer]: tag below)

[Answer]:

## Ambiguity 1: Domain Scope
Your response to Q5 ("software development") is broad. Different aspects of software development need very different policies.

### Clarification Question 2
Which aspect of software development is the primary focus?

A) Code writing and generation
B) Code review and quality assurance
C) Testing and validation
D) Full lifecycle (planning through deployment)
E) Other (please describe after [Answer]: tag below)

[Answer]:
```

#### Workflow for Clarifications

1. **Detect**: Analyze all responses for contradictions/ambiguities
2. **Create**: Generate clarification question file if issues found
3. **Inform**: Tell user about the issues and clarification file
4. **Wait**: Do not proceed until user provides clarifications
5. **Re-validate**: After clarifications, check again for consistency
6. **Proceed**: Only move forward when all contradictions are resolved

#### Example User Message
```
"I detected 2 issues in your responses:

1. Agent type vs complexity mismatch (Q1 vs Q3)
2. Domain scope ambiguity (Q5)

I've created purpose-analysis-clarification-questions.md with 2 questions to resolve these.
Please answer these clarifying questions before I can proceed with purpose analysis."
```

### Best Practices

1. **Be Specific**: Questions should be clear and unambiguous
2. **Be Comprehensive**: Cover all necessary information
3. **Be Concise**: Keep questions focused on one topic
4. **Be Practical**: Options should be realistic and actionable
5. **Be Consistent**: Use same format throughout all question files

### Phase-Specific Question Guidelines

#### DISCOVERY Phase Questions
Focus on understanding the target agent:
- What does the agent do?
- Who uses it?
- What domain does it operate in?
- How complex is the workflow?

#### DESIGN Phase Questions
Focus on architectural decisions:
- How should phases be structured?
- Which common rules apply?
- What quality mechanisms are needed?
- How should errors be handled?

#### GENERATION Phase Questions
Focus on content decisions:
- What level of detail for each file?
- Any domain-specific content to include?
- Any reference implementations to follow?

#### REFINEMENT Phase Questions
Focus on quality validation:
- Which quality dimensions need attention?
- What calibration adjustments are needed?
- Any user-specific quality requirements?

#### PACKAGING Phase Questions
Focus on plugin structure and migration:
- What plugin structure is appropriate?
- Which skills/commands should be exposed?
- Is migration from an existing structure needed?

#### Repair Loop Escalation Questions
When repair loop limits are reached, use this template:

```markdown
## Escalation: Repair Loop #{N}

The repair loop has reached its limit. Current FAIL dimension: [Dim N: Name] (Score: [value]).

### What would you like to do?

A) Continue — retry the same repair approach
B) Abort — deliver with current quality (quality note will be attached)
C) Rescope — adjust quality target or scope to resolve the issue
D) Other (please describe after [Answer]: tag below)

[Answer]:
```

## Summary

**Remember**:
- Always create question files
- Always use multiple choice format
- **Always include "Other" as the LAST option (MANDATORY)**
- Only include meaningful options — don't make up options to fill slots
- Always use [Answer]: tags
- Always wait for user completion
- Always validate responses for contradictions
- Always create clarification files if needed
- Always resolve contradictions before proceeding
- Never ask questions in chat
- Never make up options just to have A, B, C, D
- Never proceed without answers
- Never proceed with unresolved contradictions
- Never make assumptions about ambiguous responses
