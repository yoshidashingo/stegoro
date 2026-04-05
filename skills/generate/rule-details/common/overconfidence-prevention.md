# Overconfidence Prevention Guide

## Problem Statement

AI agents creating steering policies can exhibit overconfidence by:
- Not asking enough clarifying questions about the target agent's requirements
- Making assumptions about domain best practices without verification
- Generating policy files with insufficient detail based on surface-level understanding
- Skipping stages that appear unnecessary but actually add significant value
- Assuming a "standard" approach when the domain has unique requirements

This guide ensures the Steering Policy Maker asks comprehensive questions and avoids premature conclusions.

## Root Cause Analysis

Overconfidence in policy creation manifests when:

1. **Purpose Analysis**: Quickly classifying an agent type without exploring edge cases
2. **Domain Research**: Assuming well-known patterns without checking domain-specific nuances
3. **Design Phase**: Copying templates without adapting to the target agent's specific needs
4. **Generation Phase**: Producing generic content instead of domain-tailored rules
5. **Refinement Phase**: Passing quality checks without thorough comparison to AI-DLC reference
6. **Packaging Phase**: Claiming plugin completion without smoke testing all 4 agent types

---

## Solution: "When In Doubt, Ask" Philosophy

### Updated Question Generation Philosophy

**OLD APPROACH**: "Only ask questions if absolutely necessary"
**NEW APPROACH**: "When in doubt, ask the question — overconfidence leads to poor policy quality"

### Key Principles

1. **Default to Asking**: When there's any ambiguity, ask clarifying questions
2. **Comprehensive Coverage**: Evaluate ALL relevant categories, don't skip areas
3. **Thorough Analysis**: Carefully analyze ALL user responses for ambiguities
4. **Mandatory Follow-up**: Create follow-up questions for ANY unclear responses
5. **No Proceeding with Ambiguity**: Don't move forward until ALL ambiguities are resolved

---

## Phase-Specific Overconfidence Prevention

### DISCOVERY Phase

#### Purpose Analysis
- **DO NOT** classify the agent type based on the first description alone
- **DO** ask follow-up questions to confirm the classification
- **DO** present the classification with reasoning and ask for confirmation
- **DO** explore edge cases: "Are there scenarios where the agent behaves differently?"

#### Domain Research
- **DO NOT** assume domain best practices from general knowledge alone
- **DO** research domain-specific standards, regulations, and common pitfalls
- **DO** ask the user about domain-specific requirements they're aware of
- **DO** identify areas where your knowledge may be incomplete and flag them

#### Stakeholder Identification
- **DO NOT** assume a single user type without asking
- **DO** ask about all potential users, even indirect ones
- **DO** consider administrative, operational, and end-user perspectives

#### Scope Definition
- **DO NOT** estimate scope based on agent type alone
- **DO** validate scope estimates with the user
- **DO** flag areas where scope could expand and ask about boundaries

### DESIGN Phase

#### Workflow Architecture
- **DO NOT** copy a template workflow without adapting it
- **DO** design the workflow based on specific DISCOVERY findings
- **DO** ask about workflow preferences and constraints
- **DO** present multiple architecture options when viable

#### Common Rules Design
- **DO NOT** include all common rules by default without evaluation
- **DO** evaluate each common rule's relevance to the target agent
- **DO** ask about domain-specific rules that may be needed
- **DO** consider which rules need significant adaptation vs. minor customization

#### Phase Rules Design
- **DO NOT** design rules at surface level
- **DO** detail the specific content each rule file should contain
- **DO** ask about domain-specific considerations for each phase
- **DO** verify that rule files cover all identified scenarios

#### Quality Mechanisms Design
- **DO NOT** copy quality mechanisms from AI-DLC without adaptation
- **DO** evaluate which quality mechanisms are most important for the domain
- **DO** design domain-specific quality checks
- **DO** ask about quality requirements specific to the user's context

### GENERATION Phase

#### File Content Generation
- **DO NOT** generate generic placeholder content
- **DO NOT** generate content from templates alone without domain injection — domain specificity rate < 40% is a sign of insufficient injection
- **DO** generate fully detailed, domain-specific content
- **DO** reference specific domain terminology, standards, and practices
- **DO** ask for clarification when content decisions could go multiple ways
- **DO** insert minimum 2 GOOD/BAD examples per Phase Rule file

#### Integration Validation
- **DO NOT** perform superficial validation — structure tests alone are insufficient
- **DO NOT** skip content tests (Dim 12-15) even if structure tests pass
- **DO** execute all 3 test layers (structure + content + smoke)
- **DO** trace every workflow path through the generated files
- **DO** verify every cross-reference resolves
- **DO** check for logical gaps and dead ends

### REFINEMENT Phase

#### Quality Calibration
- **DO NOT** pass all dimensions automatically
- **DO NOT** use subjective assessment ("looks good enough") — use Dim 12-15 quantitative metrics with evidence
- **DO** honestly assess each dimension with specific evidence (file path:line number, count values)
- **DO** flag dimensions that need improvement
- **DO** compare with AI-DLC reference with real analysis, not superficial checks
- **DO** follow the repair judgment tree for FAIL routing — do not manually choose return targets

#### Repair Loop Discipline
- **DO NOT** silently retry if repair doesn't improve the score on 2nd attempt — question the problem classification itself
- **DO** record each repair loop in steering-state.md Repair Loop History
- **DO** escalate to user when limits are reached (max 3 total, same-target 2x)

### PACKAGING Phase

#### Plugin Structure
- **DO NOT** claim plugin completion without smoke testing — structure tests alone are insufficient
- **DO NOT** skip any of the 4 agent types (Process/Task/Analytical/Hybrid) in smoke tests
- **DO** verify SKILL.md is 75-150 lines (concise entry point, details in rule-details)
- **DO** validate all plugin.json references resolve to actual files
- **DO** check against `implementation-knowhow.md` skill design patterns

---

## Question Generation Guidelines

### For Each Stage, Evaluate ALL These Categories

#### Functional Categories
- Core functionality and behavior
- Input/output patterns
- Error scenarios and edge cases
- User interaction patterns
- Integration points

#### Non-Functional Categories
- Quality requirements
- Performance expectations
- Security considerations
- Scalability needs
- Maintainability concerns

#### Domain-Specific Categories
- Industry standards and regulations
- Common failure modes
- Best practices and anti-patterns
- Compliance requirements
- Cultural or organizational norms

### Answer Analysis Requirements

After receiving user responses, ALWAYS check for:

1. **Vague Responses**: "depends", "maybe", "not sure", "mix of", "somewhere between"
2. **Undefined Terms**: References to concepts not yet defined
3. **Contradictory Answers**: Logically inconsistent across questions
4. **Incomplete Answers**: Answers that partially address the question
5. **Implicit Assumptions**: Answers that assume context not yet established

### Follow-up Question Requirements

Create follow-up questions when:
- Any response is vague or ambiguous
- Answers conflict with each other
- New information reveals previously unknown considerations
- Domain research identifies areas not covered in initial questions
- The user's answers suggest a different agent type than initially classified

---

## Red Flags to Watch For

### Stage-Level Red Flags
- Completing Purpose Analysis without asking any questions
- Classifying agent type without presenting alternatives
- Designing workflow without domain-specific adaptations
- Generating rule files with generic content
- Passing all quality dimensions without detailed evidence

### File-Level Red Flags
- Rule files that could apply to any domain (too generic)
- Common rules copied verbatim from templates without adaptation
- Missing error handling for domain-specific scenarios
- Quality standards that don't reference domain norms
- Terminology glossary without domain-specific terms

### Workflow-Level Red Flags
- Entire DISCOVERY phase completing in one interaction
- No clarification questions created during any phase
- All stages passing quality review on first attempt
- Generated files totaling significantly less than expected line count
- No domain-specific adaptations visible in the generated policy set

---

## Success Indicators

- Appropriate number of clarifying questions for domain complexity
- Thorough analysis of user responses with follow-up when needed
- Clear, domain-specific content in all generated rule files
- Honest quality calibration with specific evidence for each dimension
- User feedback confirms the policy set accurately reflects their needs

## Key Takeaway

**It's better to ask too many questions than to make incorrect assumptions about the target agent's domain and needs.** The cost of asking clarifying questions upfront is far less than the cost of generating a policy set that doesn't accurately guide the target agent.

**Remember**: A steering policy set that makes wrong assumptions will propagate those errors to every interaction the target agent has with its users.
