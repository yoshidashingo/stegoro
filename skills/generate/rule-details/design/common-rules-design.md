# Common Rules Design — DESIGN Phase

## Purpose
Select which standard common rules apply to the target agent, identify domain-specific common rules needed, and design the adaptations required for each. This ensures the cross-phase rules are tailored to the target agent's domain rather than being generic copies.

## Prerequisites
- All DISCOVERY phase artifacts available and approved
- Workflow Architecture completed and approved
- Common rules template set understood (10 standard common rules)

## Execution Classification
**Type**: ALWAYS
This stage always executes as common rules are fundamental to every steering policy set.

---

## Execution Steps

### Step 1: Load Prior Context
**Action**: Load all relevant prior artifacts
**Input**: DISCOVERY artifacts + Workflow Architecture
**Output**: Design context for common rules

Key context:
- Domain terminology from domain research
- Quality standards from domain research
- Interaction patterns from stakeholder analysis
- Phase/stage structure from workflow architecture
- Quality target from scope definition

### Step 2: Evaluate Standard Common Rules
**Action**: Assess each standard common rule's relevance and needed adaptation
**Input**: Standard common rules template + design context
**Output**: Common rules evaluation matrix

#### Standard Common Rules Evaluation

For each of the 10 standard common rules, determine:

| Rule | Include? | Adaptation Level | Notes |
|------|----------|-----------------|-------|
| `welcome-message.md` | ALWAYS | Heavy | Must be fully customized for target agent |
| `process-overview.md` | ALWAYS | Heavy | Must reflect target agent's workflow |
| `question-format-guide.md` | ALWAYS | Light-Medium | Format is standard, examples need adaptation |
| `content-validation.md` | ALWAYS | Medium | Add domain-specific validation rules |
| `session-continuity.md` | ALWAYS | Medium | State file format needs customization |
| `error-handling.md` | ALWAYS | Heavy | Phase-specific errors are all domain-specific |
| `overconfidence-prevention.md` | ALWAYS | Medium | Domain-specific overconfidence scenarios |
| `terminology.md` | ALWAYS | Heavy | Domain terminology is entirely custom |
| `quality-standards.md` | ALWAYS | Medium-Heavy | Quality dimensions adapted to domain |
| `output-structure-patterns.md` | ALWAYS | Medium | Patterns adapted to phase structure |

#### Adaptation Levels

- **Light**: Mostly template content with minor domain examples
- **Medium**: Template structure with significant domain content
- **Heavy**: Structure inspired by template but content is largely domain-specific

### Step 3: Design Welcome Message
**Action**: Design the content for the target agent's welcome message
**Input**: Agent purpose + workflow architecture
**Output**: Welcome message design specification

The welcome message should include:
1. **Agent introduction**: What the agent does and how it helps
2. **Workflow overview**: ASCII diagram of the phase/stage structure
3. **Phase descriptions**: Brief purpose of each phase
4. **Key principles**: The agent's guiding principles
5. **Getting started**: What happens first and what the user should do

### Step 4: Design Question Format Adaptations
**Action**: Design how the question format guide applies to this domain
**Input**: Domain characteristics + stage structure
**Output**: Question format adaptation notes

Consider:
- Are there domain-specific question categories?
- Do question files need domain-specific naming conventions?
- Are there domain-specific contradiction patterns to detect?
- What are typical domain-specific clarification scenarios?

### Step 5: Design Content Validation Adaptations
**Action**: Design domain-specific content validation rules
**Input**: Domain research + output types
**Output**: Content validation adaptation notes

Consider:
- What types of content will the target agent generate?
- Are there domain-specific syntax or formatting requirements?
- What visual content (diagrams, charts) might be generated?
- What fallback patterns are needed for domain content?

### Step 6: Design Error Handling Adaptations
**Action**: Design phase-specific error handling for the target agent
**Input**: Workflow architecture + domain pitfalls
**Output**: Error handling design specification

For each phase in the target agent's workflow:
1. List possible error scenarios
2. Classify severity (Critical/High/Medium/Low)
3. Define recovery procedures
4. Identify escalation points

### Step 7: Design Terminology Glossary
**Action**: Design the target agent's terminology glossary
**Input**: Domain terminology from research + agent-specific terms
**Output**: Terminology design specification

Include:
- Phase/stage hierarchy terminology
- Domain-specific terms
- Agent-specific concepts
- Execution classifications
- Artifact types
- Common abbreviations

### Step 8: Design Session Continuity
**Action**: Design how session resumption works for the target agent
**Input**: Workflow architecture + state tracking design
**Output**: Session continuity design specification

Define:
- State file format and update rules
- Welcome back prompt template
- Context loading priorities per phase
- Artifact loading requirements per stage

### Step 9: Identify Domain-Specific Common Rules
**Action**: Determine if additional common rules are needed beyond the standard 10
**Input**: Domain research + workflow architecture
**Output**: Domain-specific common rules list

Domain-specific common rules might include:
- **Domain compliance rules**: Regulatory or standards compliance
- **Domain output standards**: Specific output format requirements
- **Domain quality checks**: Domain-specific quality validation
- **Domain collaboration rules**: Multi-user interaction patterns
- **Domain security rules**: Security or access control requirements

### Step 10: Generate Common Rules Design Questions (If Needed)
**Action**: Create questions about common rules design decisions
**Input**: All design work with decision points
**Output**: Common rules design question file

### Step 11: Present Common Rules Design
**Action**: Present complete common rules design for approval
**Input**: All design outputs
**Output**: Formatted design presentation

---

## Output Artifacts

### Common Rules Design Document
- **File**: `steering-docs/<agent-name>/design/common-rules-design.md`
- **Content**: Complete common rules inventory with adaptation specifications
- **Format**:

```markdown
# Common Rules Design

## Rules Inventory
| # | Rule File | Adaptation Level | Key Changes |
|---|-----------|-----------------|-------------|
| 1 | welcome-message.md | Heavy | [Summary] |
| 2 | process-overview.md | Heavy | [Summary] |
| 3 | question-format-guide.md | Light | [Summary] |
| 4 | content-validation.md | Medium | [Summary] |
| 5 | session-continuity.md | Medium | [Summary] |
| 6 | error-handling.md | Heavy | [Summary] |
| 7 | overconfidence-prevention.md | Medium | [Summary] |
| 8 | terminology.md | Heavy | [Summary] |
| 9 | quality-standards.md | Medium | [Summary] |
| 10 | output-structure-patterns.md | Medium | [Summary] |
| 11+ | [domain-specific] | N/A | [Description] |

## Detailed Adaptation Specifications

### 1. Welcome Message
- **Agent Name**: [name]
- **Introduction**: [key message]
- **Workflow Diagram**: [based on architecture]
- **Principles**: [list]

### 2. Process Overview
[Adaptation details]

[Continue for each rule]

## Domain-Specific Common Rules
[If any identified, include design specifications]

## Cross-Rule Dependencies
[How common rules reference each other]
```

---

## Completion Message

### REVIEW REQUIRED

**Common Rules Design is complete.** Here's a summary:

- **Standard Rules**: 10 (all included with varying adaptation levels)
- **Domain-Specific Rules**: [N] additional rules identified
- **Total Common Rules**: [N]
- **Heavy Adaptations**: [list]
- **Key Domain Additions**: [list]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the common rules design based on your feedback
**B) Continue to Phase Rules Design** — Proceed to design phase-specific rule files

---

## Error Handling

### Standard Rule Doesn't Apply
- **Issue**: A standard common rule seems irrelevant to the domain
- **Solution**: Evaluate carefully — it may apply in a different way. If truly irrelevant, document why and confirm with user
- **Note**: All 10 standard rules are almost always applicable; if one seems irrelevant, consider whether the domain has an analogous need

### Too Many Domain-Specific Rules
- **Issue**: Domain research suggests many additional common rules
- **Solution**: Evaluate if some can be merged or are actually phase-specific rules
- **Workaround**: Limit to the most critical domain rules, note others for future addition

### Conflicting Design Requirements
- **Issue**: Domain requirements conflict with standard rule patterns
- **Solution**: Prioritize domain requirements, adapt standard patterns to fit
- **Recovery**: Create custom rule that fulfills both standard and domain needs

## References
- `design/workflow-architecture.md` — Phase/stage structure
- `discovery/domain-research.md` — Domain terminology and practices
- `discovery/scope-definition.md` — Quality target
- `common/output-structure-patterns.md` — Standard file patterns
