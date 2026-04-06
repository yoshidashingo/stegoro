# Purpose Analysis — DISCOVERY Phase

## Purpose
Analyze the user's stated purpose to understand what AI agent needs steering policies, classify the agent type, and establish a clear foundation for all subsequent phases.

## Prerequisites
- Welcome message has been displayed (from `common/welcome-message.md`)
- Common rules loaded at workflow start
- User has provided initial purpose statement

## Execution Classification
**Type**: ALWAYS
This stage always executes as it establishes the fundamental understanding of the target agent.

## Adaptive Depth
- **Minimal**: Purpose is very clear, well-known agent pattern (e.g., "code reviewer")
- **Standard**: Purpose requires some clarification, agent has unique aspects
- **Comprehensive**: Purpose is novel, domain is complex, agent combines multiple patterns

---

## Execution Steps

### Step 1: Capture Initial Purpose Statement
**Action**: Record the user's exact words describing their target agent
**Input**: User's initial message/request
**Output**: Raw purpose statement logged in audit.md
**Validation**: Verify the statement has been captured completely without summarization

**MANDATORY**: Log the complete raw user input in audit.md with ISO 8601 timestamp.

### Step 2: Intent Analysis
**Action**: Analyze the purpose statement for key indicators
**Input**: Raw purpose statement
**Output**: Intent analysis document

Analyze along these dimensions:
1. **Core Function**: What does the agent fundamentally do?
2. **Domain**: What field/industry/area does it operate in?
3. **Users**: Who will interact with the agent?
4. **Scope**: How broad or narrow is the agent's responsibility?
5. **Complexity Indicators**: What suggests simple vs complex workflow needs?
6. **Quality Indicators**: What suggests high vs low quality requirements?
7. **Risk Profile**: What could go wrong if the agent behaves incorrectly?

### Step 3: Agent Type Classification
**Action**: Classify the target agent into one of four types
**Input**: Intent analysis results
**Output**: Preliminary classification with reasoning

#### Classification Matrix

| Indicator | Process Agent | Task Agent | Analytical Agent | Hybrid Agent |
|-----------|--------------|------------|-----------------|--------------|
| **Workflow** | Multi-phase, sequential | Single task, bounded | Input→Process→Output | Mixed patterns |
| **User Interaction** | Many checkpoints | Few checkpoints | Input + review output | Context-dependent |
| **State Management** | Complex state tracking | Minimal state | Moderate state | Variable state |
| **Duration** | Long-running | Short/medium | Medium | Variable |
| **Typical Phases** | 3-5 phases | 2-3 phases | 2-4 phases | 3-6 phases |
| **Examples** | SDLC, Audit process | Code review, Linting | Doc gen, Data analysis | DevOps, PM assistant |

#### Classification Process

1. **Score each type** based on how well the purpose matches:
   - Strong Match (3): Multiple indicators clearly match
   - Partial Match (2): Some indicators match, others unclear
   - Weak Match (1): Few indicators match
   - No Match (0): Purpose doesn't align

2. **Determine primary classification** based on highest score

3. **Check for Hybrid**: If two or more types score 2+, consider Hybrid classification

4. **Present classification with reasoning** to user for confirmation

### Step 4: Core Capabilities Identification
**Action**: Identify the specific capabilities the target agent must have
**Input**: Intent analysis + agent type classification
**Output**: Capabilities list

Identify capabilities in these categories:
- **Primary Capabilities**: What the agent MUST do (core functionality)
- **Secondary Capabilities**: What the agent SHOULD do (supporting functionality)
- **Quality Capabilities**: How the agent ensures quality (validation, checking)
- **Interaction Capabilities**: How the agent communicates with users
- **Error Capabilities**: How the agent handles problems

### Step 5: Complexity Assessment
**Action**: Determine the overall complexity level
**Input**: All previous analysis
**Output**: Complexity classification

#### Complexity Factors

| Factor | Simple | Standard | Complex |
|--------|--------|----------|---------|
| **Phases** | 2 | 3-4 | 5+ |
| **Stages per Phase** | 2-3 | 3-5 | 5+ |
| **Conditional Logic** | Minimal | Moderate | Extensive |
| **User Types** | 1 | 2-3 | 4+ |
| **Domain Risk** | Low | Medium | High |
| **Quality Requirements** | Basic | Standard | Stringent |
| **Error Scenarios** | Few | Moderate | Many |
| **State Management** | Minimal | Moderate | Complex |

Assign overall complexity: **Simple** / **Standard** / **Complex**

#### Quantitative Indicators by Complexity

| Indicator | Simple | Standard | Complex |
|-----------|--------|----------|---------|
| Expected phases | 2-3 | 3-4 | 4-5 |
| Expected stages | 6-10 | 10-14 | 14-18 |
| Quality dimensions | 11 (standard) | 11-15 | 15 (all) |
| core-workflow lines | 200-300 | 300-450 | 450-600 |
| Total policy files | ~15 | ~25 | ~30-45 |

#### 4-Type Analysis Template

For each agent type, verify these type-specific characteristics:

| Check | Process | Task | Analytical | Hybrid |
|-------|---------|------|-----------|--------|
| Sequential phases? | Yes (3-5) | Limited (2-3) | Variable (2-4) | Mixed |
| State tracking? | Heavy | Light | Medium | Heavy |
| Approval gates? | Every stage | Key stages | Output stages | Context-dependent |
| Checkpoint pattern | Plan→Approve→Execute→Verify | Analyze→Process→Report | Ingest→Transform→Produce | Mode-switching |

### Step 6: Generate Purpose Analysis Questions (If Needed)
**Action**: Create clarification questions if purpose is ambiguous
**Input**: All analysis so far
**Output**: Purpose analysis question file (if needed)

**IMPORTANT**: Follow the overconfidence prevention guide (`common/overconfidence-prevention.md`). When in doubt, ASK.

Create `steering-docs/<agent-name>/discovery/purpose-analysis-questions.md` if ANY of these are true:
- Agent type classification is not clearly one type (score tie or close scores)
- Core capabilities list has uncertainties
- Complexity assessment has mixed signals
- Domain is unfamiliar or specialized
- User's purpose statement was brief or ambiguous

#### Question Categories for Purpose Analysis
- Agent's primary function and workflow
- Target users and interaction patterns
- Domain-specific requirements
- Quality and compliance needs
- Error tolerance and risk profile
- Integration with other systems
- Scalability expectations

### Step 7: Present Purpose Analysis Summary
**Action**: Present findings to user for approval
**Input**: All analysis results
**Output**: Formatted summary for user review

---

## Question Generation

### When to Ask Questions
- **ALWAYS ask** if complexity assessment shows mixed signals
- **ALWAYS ask** if agent type classification is ambiguous
- **ALWAYS ask** if the domain is unfamiliar or specialized
- **Consider asking** if the purpose statement was brief (under 2 sentences)
- **Consider asking** if multiple reasonable interpretations exist

### Question File Template

```markdown
# Purpose Analysis Questions

Based on your description, I have some questions to ensure I understand your target agent correctly.

## Question 1
What is the primary workflow pattern for your target agent?

A) Multi-step sequential process with checkpoints (like a development lifecycle)
B) Single focused task with preparation and reporting (like a code reviewer)
C) Data/content processing pipeline (like a document generator)
D) Multiple operational modes depending on context (like a DevOps assistant)
E) Other (please describe after [Answer]: tag below)

[Answer]:

## Question 2
[Additional questions based on analysis gaps]
```

---

## Output Artifacts

### Purpose Analysis Summary
- **File**: `steering-docs/<agent-name>/discovery/purpose-analysis-summary.md`
- **Content**: Agent classification, capabilities, complexity assessment, reasoning
- **Format**:

```markdown
# Purpose Analysis Summary

## Target Agent
- **Name**: [agent name from user]
- **Description**: [user's purpose statement]

## Agent Classification
- **Type**: [Process / Task / Analytical / Hybrid]
- **Confidence**: [High / Medium / Low]
- **Reasoning**: [Why this classification was chosen]

## Core Capabilities
### Primary
- [Capability 1]
- [Capability 2]

### Secondary
- [Capability 1]
- [Capability 2]

### Quality
- [Capability 1]

## Complexity Assessment
- **Overall**: [Simple / Standard / Complex]
- **Estimated Phases**: [N]
- **Estimated Stages**: [N]
- **Key Complexity Drivers**: [List]

## Domain Profile
- **Domain**: [domain name]
- **Familiarity**: [Well-known / Moderate / Specialized]
- **Risk Level**: [Low / Medium / High]

## Open Questions
- [Any remaining questions for later phases]
```

---

## Completion Message

### REVIEW REQUIRED

**Purpose Analysis is complete.** Here's a summary of what was determined:

- **Target Agent**: [agent name] — [brief description]
- **Agent Type**: [classification] ([confidence level] confidence)
- **Complexity**: [level]
- **Estimated Scope**: [N phases, N stages]

Key findings:
- [Finding 1]
- [Finding 2]
- [Finding 3]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the purpose analysis based on your feedback
**B) Continue to Domain Research** — Proceed to research domain-specific best practices

---

## Error Handling

### Ambiguous Purpose Statement
- **Issue**: User's description could apply to multiple agent types
- **Solution**: Create purpose-analysis-questions.md with classification questions
- **Do Not Proceed**: Until classification is confirmed

### Novel Agent Pattern
- **Issue**: Purpose doesn't clearly match any standard classification
- **Solution**: Present Hybrid classification, ask user which aspects are most important
- **Fallback**: Use the closest matching type, note deviations for later adaptation

### Changing Purpose
- **Issue**: User changes their purpose after analysis starts
- **Solution**: Log the change in audit.md, restart from Step 1 with new purpose
- **Preserve**: Any domain research already done (may still be relevant)

### Insufficient Information
- **Issue**: User provides very minimal description
- **Solution**: Create comprehensive question file covering all classification dimensions
- **Do Not Proceed**: Until sufficient information is gathered to classify the agent

## References
- `common/terminology.md` — Agent type definitions
- `common/overconfidence-prevention.md` — Question generation philosophy
- `common/question-format-guide.md` — Question file format
