# Quality Mechanisms Design — DESIGN Phase

## Purpose
Design the quality assurance mechanisms that will be embedded in the target agent's steering policy set. These mechanisms ensure the target agent maintains consistent quality throughout its operation, mapped to the 15 quality dimensions (11 AI-DLC standard + 4 domain-specific).

## Prerequisites
- All DISCOVERY phase artifacts available and approved
- Workflow Architecture completed and approved
- Common Rules Design completed and approved
- Phase Rules Design completed and approved

## Execution Classification
**Type**: ALWAYS
This stage always executes as quality mechanisms are essential to every steering policy set.

---

## Execution Steps

### Step 1: Load All Design Context
**Action**: Load all prior DISCOVERY and DESIGN artifacts
**Input**: All phase artifacts
**Output**: Complete design context for quality mechanisms

### Step 2: Design Checkpoint System
**Action**: Design the two-level checkpoint system for the target agent
**Input**: Workflow architecture + phase rules design
**Output**: Checkpoint system specification

#### Plan-Level Checkpoints
- **Purpose**: Track detailed execution progress within each stage
- **Implementation**: Checkbox lists in plan files
- **Update Rule**: Mark checkbox immediately after completing step (same interaction)

Design for each stage:
- What plan steps need tracking?
- What granularity of checkboxes is appropriate?
- How do plan checkpoints map to stage completion?

#### Stage-Level Checkpoints
- **Purpose**: Track overall workflow progress
- **Implementation**: State file with stage completion status
- **Update Rule**: Update state file when stage status changes

Design:
- State file format (see session-continuity design)
- Status transitions: PENDING → IN PROGRESS → COMPLETED (or SKIPPED)
- Timestamp requirements
- Decision logging format

### Step 3: Design Audit Trail System
**Action**: Design the audit logging mechanism for the target agent
**Input**: Workflow architecture + interaction patterns
**Output**: Audit trail specification

#### Audit Trail Requirements
1. **Log Every User Input**: Complete raw text, never summarized
2. **Log Every AI Response**: Action taken, decision made
3. **Timestamp Format**: ISO 8601 (YYYY-MM-DDTHH:MM:SSZ)
4. **Stage Context**: Current phase, stage, and step
5. **Append-Only**: Never overwrite existing entries

#### Audit Log Design

```markdown
# [Agent Name] — Audit Log

## [Stage Name]
**Timestamp**: [ISO timestamp]
**User Input**: "[Complete raw input]"
**AI Response**: "[Response or action]"
**Context**: [Stage and step context]

---
```

#### Domain-Specific Audit Considerations
- Are there regulatory requirements for audit logging?
- What level of detail does the domain require?
- Are there retention requirements?
- Do different stakeholders need different audit views?

### Step 4: Design Content Validation Rules
**Action**: Design validation rules specific to the target agent's content types
**Input**: Domain research + output types
**Output**: Content validation specification

#### Content Types to Validate
For each type of content the target agent will generate:
1. **Content Type**: What kind of content (markdown, code, diagrams, etc.)
2. **Validation Rules**: What to check before writing
3. **Common Errors**: What typically goes wrong
4. **Fallback Strategy**: What to do if validation fails

#### Domain-Specific Validation
- What domain-specific content formats need validation?
- Are there domain-specific syntax requirements?
- What domain-specific quality checks apply?

### Step 5: Design Overconfidence Prevention Measures
**Action**: Design domain-specific overconfidence prevention for the target agent
**Input**: Domain pitfalls + stage designs
**Output**: Overconfidence prevention specification

For each phase of the target agent:
1. **Overconfidence Risk Areas**: Where might the agent skip important questions?
2. **Mandatory Question Triggers**: When MUST questions be asked?
3. **Red Flags**: What behaviors indicate overconfidence?
4. **Prevention Measures**: Specific rules to prevent overconfidence

#### Domain-Specific Overconfidence Patterns
- What domain mistakes commonly result from assumptions?
- What appears simple in this domain but actually requires clarification?
- What domain knowledge gaps should trigger questions?

### Step 6: Design Error Recovery Framework
**Action**: Design the error handling and recovery framework
**Input**: Phase rules design + domain failure modes
**Output**: Error recovery specification

#### Error Categories
For each phase of the target agent:
1. **Critical Errors**: What blocks the workflow entirely?
2. **High Errors**: What prevents a phase from completing?
3. **Medium Errors**: What can be worked around?
4. **Low Errors**: What is cosmetic or non-blocking?

#### Recovery Procedures
For each error category:
1. **Detection**: How is the error identified?
2. **Communication**: How is the user informed?
3. **Resolution Options**: What can be done to fix it?
4. **Escalation**: When should the user be asked for help?
5. **Documentation**: How is the error logged?

### Step 7: Design Depth Level Adaptation
**Action**: Design how the target agent adapts detail level based on complexity
**Input**: Agent type + domain complexity factors
**Output**: Depth adaptation specification

#### Depth Level Definitions

| Level | When | Characteristics |
|-------|------|----------------|
| **Minimal** | [Simple scenarios] | [What minimal looks like] |
| **Standard** | [Normal scenarios] | [What standard looks like] |
| **Comprehensive** | [Complex scenarios] | [What comprehensive looks like] |

#### Depth Selection Criteria
- What factors determine depth in this domain?
- How does depth affect each stage?
- What is the minimum acceptable depth?
- What triggers escalation to higher depth?

### Step 8: Design Completion Message System
**Action**: Design the standardized completion message format
**Input**: Phase/stage structure + approval gates
**Output**: Completion message specification

#### Standard 2-Option Format

```markdown
### REVIEW REQUIRED
**[Stage Name] is complete.** [Summary of what was produced]

### WHAT'S NEXT
**A) Request Changes** — [What happens if they request changes]
**B) Continue to [Next Stage]** — [What happens if they continue]
```

#### Phase Completion Format
```markdown
### PHASE COMPLETE: [Phase Name]
**All stages in [Phase Name] are complete.**
[Summary of phase outputs]

**A) Proceed to [Next Phase]** — Continue
**B) Review [Current Phase] outputs** — Review first
```

#### Workflow Completion Format
```markdown
### WORKFLOW COMPLETE
**[Agent Name] has completed its workflow.**
[Final deliverables summary]
[Next steps for the user]
```

### Step 9: Map to 15 Quality Dimensions
**Action**: Verify all 15 quality dimensions are covered in the design
**Input**: All quality mechanism designs
**Output**: Quality dimension coverage map

| Dimension | Covered By | Design Location |
|-----------|------------|-----------------|
| 1. Adaptive Workflow | Workflow Architecture + Stage Classifications | workflow-architecture.md |
| 2. Mandatory Checkpoints | Checkpoint System Design | Step 2 above |
| 3. Question File Format | Question Format Adaptations | common-rules-design.md |
| 4. Content Validation | Content Validation Rules | Step 4 above |
| 5. Audit Trail | Audit Trail System | Step 3 above |
| 6. Error Handling | Error Recovery Framework | Step 6 above |
| 7. Overconfidence Prevention | Prevention Measures | Step 5 above |
| 8. Depth Levels | Depth Adaptation | Step 7 above |
| 9. Session Continuity | State Tracking + Session Design | workflow-architecture.md + common-rules-design.md |
| 10. Terminology Standardization | Terminology Glossary Design | common-rules-design.md |
| 11. Standardized Completion Messages | Message System Design | Step 8 above |
| 12. Domain Specificity Rate | Domain Content Injection + Measurement | phase-rules-design.md (G3) |
| 13. Example Coverage | GOOD/BAD Example Injection | phase-rules-design.md (G3) |
| 14. Artifact Template Completeness | Output Template Design | phase-rules-design.md (G2, G3) |
| 15. Pitfall Reference Rate | Domain Pitfall Integration | phase-rules-design.md (G3) |

**VERIFY**: All 15 dimensions have explicit design coverage. If any dimension lacks coverage, add design specifications before proceeding.

### Step 10: Generate Quality Mechanisms Questions (If Needed)
**Action**: Create questions about quality design decisions
**Input**: All design work with decision points
**Output**: Quality mechanisms question file

### Step 11: Present Quality Mechanisms Design
**Action**: Present complete quality mechanisms design for approval
**Input**: All design outputs
**Output**: Formatted design presentation

---

## Output Artifacts

### Quality Mechanisms Design Document
- **File**: `steering-docs/<agent-name>/design/quality-mechanisms-design.md`
- **Content**: Complete quality mechanisms specification
- **Format**:

```markdown
# Quality Mechanisms Design

## Summary
- **Checkpoint Levels**: 2 (Plan-level + Stage-level)
- **Audit Trail**: [Format description]
- **Content Validation**: [N] content types covered
- **Error Categories**: [N] per phase
- **Depth Levels**: 3 (Minimal/Standard/Comprehensive)
- **Quality Dimensions Covered**: 15/15

## Checkpoint System Design
[Detailed specification]

## Audit Trail Design
[Detailed specification]

## Content Validation Design
[Detailed specification]

## Overconfidence Prevention Design
[Detailed specification]

## Error Recovery Design
[Detailed specification]

## Depth Adaptation Design
[Detailed specification]

## Completion Message Design
[Detailed specification]

## Quality Dimension Coverage
[11-dimension mapping with evidence]
```

---

## Completion Message

### REVIEW REQUIRED

**Quality Mechanisms Design is complete.** Here's a summary:

- **Quality Dimensions Covered**: 15/15
- **Checkpoint System**: 2-level (plan + stage)
- **Error Categories**: [N] per phase
- **Content Validation**: [N] content types
- **Depth Levels**: 3 (Minimal/Standard/Comprehensive)

Key quality design decisions:
- [Decision 1]
- [Decision 2]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the quality mechanisms design based on your feedback
**B) Continue to GENERATION Phase** — Proceed to generate the actual policy files

---

### PHASE COMPLETE: DESIGN

**All stages in the DESIGN phase are complete.**

**Summary of outputs:**
- Workflow Architecture: [N] phases, [N] stages designed
- Common Rules Design: [N] rules with adaptation specs
- Phase Rules Design: [N] stage files designed
- Quality Mechanisms Design: 15/15 dimensions covered

### WHAT'S NEXT

The next phase is **GENERATION**: Create the actual steering policy files.

**A) Proceed to GENERATION Phase** — Continue the workflow
**B) Review DESIGN outputs** — Review before moving forward

---

## 15 Quality Dimensions Design Guide

When designing quality mechanisms, plan for 15 dimensions (11 standard + 4 domain-specific):

### Domain-Specific Dimensions Design Template

| # | Dimension | What to Design | Measurement Method | Threshold |
|---|-----------|---------------|-------------------|-----------|
| 12 | Domain Specificity Rate | How to measure domain-specific lines per file | Line-level assessment against terminology.md | >= 40% |
| 13 | Example Coverage | How many examples per file, GOOD/BAD format | Count example blocks per file | >= 2/file |
| 14 | Artifact Template Completeness | Which files need output templates | Files with templates ÷ files needing templates | = 100% |
| 15 | Pitfall Reference Rate | How error handling cites domain pitfalls | Error items citing pitfalls ÷ total error items | >= 50% |

### 3-Layer Automated Testing Design

Design automated testing for execution at G4 and P2:

| Layer | G4 (Policy) | P2 (Plugin) |
|-------|-------------|-------------|
| Structure | File existence, references, Markdown, flow paths | plugin.json fields, file existence, directory structure |
| Content | Dim 12-15 pre-validation | Line count ranges (SKILL.md 75-150, etc.) |
| Smoke | 4-type flow traversal on policy | 4-type flow traversal on plugin |

---

## Error Handling

### Quality Dimension Not Covered
- **Issue**: One or more of the 15 quality dimensions lacks design coverage
- **Solution**: Add design specifications for the missing dimension
- **Do Not Proceed**: Until all 15 dimensions have explicit coverage

### Design Conflicts Between Dimensions
- **Issue**: Quality mechanisms conflict with each other (e.g., depth vs efficiency)
- **Solution**: Prioritize based on domain risk, present trade-offs to user
- **Recovery**: Define clear precedence rules for when mechanisms conflict

### Over-Designed Quality Mechanisms
- **Issue**: Quality mechanisms are too heavy for the target agent's needs
- **Solution**: Scale back to match the quality target from scope definition
- **Workaround**: Implement core mechanisms, make advanced ones optional

### Under-Designed Quality Mechanisms
- **Issue**: Quality mechanisms insufficient for the domain's risk profile
- **Solution**: Enhance mechanisms in areas where domain risk is highest
- **Recovery**: May need to return to scope definition to adjust quality target

## References
- `design/workflow-architecture.md` — Phase/stage structure
- `design/common-rules-design.md` — Common rules adaptations
- `design/phase-rules-design.md` — Stage rule specifications
- `common/quality-standards.md` — AI-DLC quality benchmark
- `discovery/domain-research.md` — Domain risk and quality requirements
