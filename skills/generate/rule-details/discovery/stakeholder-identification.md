# Stakeholder Identification — DISCOVERY Phase

## Purpose
Identify all user types and stakeholders who will interact with the target agent, map their needs and interaction patterns, and ensure the generated steering policies accommodate all perspectives. This ensures no user group is overlooked in the policy design.

## Prerequisites
- Purpose Analysis completed and approved
- Domain Research completed and approved
- Agent type classification confirmed

## Execution Classification
**Type**: CONDITIONAL

**Execute IF**:
- Multiple user types or personas will interact with the target agent
- Different stakeholders have different needs, priorities, or perspectives
- The target agent serves cross-functional teams
- Access control, role-based behavior, or permission levels are needed
- Domain research revealed multiple interaction patterns
- Purpose analysis identified more than one user category

**Skip IF**:
- Single user type with clear, unified needs
- Simple task agent with one straightforward interaction pattern
- User explicitly confirms single stakeholder type
- Agent is internal tool used by homogeneous group

---

## Execution Steps

### Step 1: Initial Stakeholder Inventory
**Action**: Identify all potential stakeholders from prior analysis
**Input**: Purpose analysis + domain research
**Output**: Preliminary stakeholder list

Sources for stakeholder identification:
1. **Purpose statement**: Who did the user mention?
2. **Domain research**: Who typically uses tools in this domain?
3. **Agent type**: What interaction patterns does this type imply?
4. **Implicit stakeholders**: Who else might be affected?

### Step 2: Stakeholder Classification
**Action**: Classify each stakeholder by their relationship to the agent
**Input**: Preliminary stakeholder list
**Output**: Classified stakeholder map

#### Stakeholder Categories

| Category | Description | Policy Impact |
|----------|-------------|--------------|
| **Primary Users** | Direct, frequent interaction with the agent | Full workflow support needed |
| **Secondary Users** | Occasional interaction, specific tasks | Simplified workflow paths |
| **Reviewers** | Review agent outputs, approve decisions | Approval gates, review formats |
| **Administrators** | Configure, maintain, or customize the agent | Configuration and maintenance rules |
| **Beneficiaries** | Indirectly benefit from agent outputs | Output format and quality requirements |

### Step 3: Needs Analysis Per Stakeholder
**Action**: Map specific needs for each stakeholder type
**Input**: Classified stakeholder map
**Output**: Stakeholder needs matrix

For each stakeholder:
- **Goals**: What do they want to achieve?
- **Pain Points**: What problems should the agent solve?
- **Interaction Frequency**: How often do they interact?
- **Technical Level**: How sophisticated is their usage?
- **Quality Expectations**: What quality standards matter to them?
- **Information Needs**: What outputs do they need from the agent?

### Step 4: Interaction Pattern Design
**Action**: Define how each stakeholder type interacts with the agent
**Input**: Stakeholder needs analysis
**Output**: Interaction pattern document

For each stakeholder:
- **Entry Point**: How do they start interacting?
- **Typical Workflow**: What is their common path through the agent?
- **Decision Points**: Where do they need to make choices?
- **Approval Authority**: What can they approve or reject?
- **Output Consumption**: How do they receive and use outputs?
- **Error Handling**: What happens when things go wrong for them?

### Step 5: Conflict Identification
**Action**: Identify conflicting requirements between stakeholders
**Input**: All stakeholder analysis
**Output**: Conflict report

Look for:
- **Priority Conflicts**: Stakeholders wanting different things prioritized
- **Quality Conflicts**: Different quality standards or acceptable trade-offs
- **Workflow Conflicts**: Stakeholders preferring different process flows
- **Output Conflicts**: Stakeholders needing different output formats
- **Access Conflicts**: Disagreements about who should see/do what

### Step 6: Resolution Strategy
**Action**: Propose resolution for identified conflicts
**Input**: Conflict report
**Output**: Resolution strategy

Resolution approaches:
1. **Role-Based Paths**: Different workflow branches per stakeholder type
2. **Configurable Outputs**: Adaptable output formats
3. **Priority Framework**: Clear prioritization rules
4. **Escalation Paths**: How to handle unresolvable conflicts

### Step 7: Generate Stakeholder Questions (If Needed)
**Action**: Create questions to validate stakeholder analysis
**Input**: All analysis with uncertainties
**Output**: Stakeholder question file

Create `steering-docs/<agent-name>/discovery/stakeholder-identification-questions.md` if:
- Stakeholder list may be incomplete
- Interaction patterns are unclear
- Conflicts need user input to resolve
- Access control requirements are uncertain

### Step 8: Present Stakeholder Map
**Action**: Present complete stakeholder analysis for approval
**Input**: All analysis results
**Output**: Formatted stakeholder map for user review

---

## Output Artifacts

### Stakeholder Map
- **File**: `steering-docs/<agent-name>/discovery/stakeholder-map.md`
- **Content**: Complete stakeholder analysis with interaction patterns
- **Format**:

```markdown
# Stakeholder Map

## Stakeholder Summary
| Stakeholder | Category | Frequency | Key Need |
|-------------|----------|-----------|----------|
| [Type 1] | Primary | Daily | [Need] |
| [Type 2] | Reviewer | Weekly | [Need] |
| [Type 3] | Admin | Monthly | [Need] |

## Detailed Stakeholder Profiles

### [Stakeholder Type 1]: [Name]
- **Category**: [Primary/Secondary/Reviewer/Admin/Beneficiary]
- **Goals**: [What they want to achieve]
- **Interaction Pattern**: [How they use the agent]
- **Quality Expectations**: [What quality means to them]
- **Policy Implications**: [How this affects policy design]

### [Stakeholder Type 2]: [Name]
[Continue for each stakeholder]

## Interaction Patterns
[Diagram or description of how different stakeholders interact]

## Identified Conflicts
### Conflict 1: [Title]
- **Stakeholders**: [Type A] vs [Type B]
- **Issue**: [What conflicts]
- **Resolution**: [Proposed resolution]

## Policy Design Implications
1. [Implication 1 — how stakeholder needs affect policy design]
2. [Implication 2]
3. [Implication 3]
```

---

## Completion Message

### REVIEW REQUIRED

**Stakeholder Identification is complete.** Here's a summary:

- **Stakeholder Types Identified**: [N]
  - Primary Users: [N]
  - Secondary Users: [N]
  - Reviewers: [N]
  - Administrators: [N]
- **Interaction Patterns Defined**: [N]
- **Conflicts Identified**: [N] (with proposed resolutions)

Key findings:
- [Finding 1]
- [Finding 2]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the stakeholder analysis based on your feedback
**B) Continue to Scope Definition** — Proceed to define the policy set's scope and boundaries

---

## Error Handling

### Too Many Stakeholders
- **Issue**: Number of stakeholder types makes policy design impractical
- **Solution**: Propose stakeholder grouping, merge similar types
- **Workaround**: Focus on primary and secondary users, simplify others

### Conflicting Requirements Can't Be Resolved
- **Issue**: Stakeholder needs are fundamentally incompatible
- **Solution**: Escalate to user, ask which stakeholder to prioritize
- **Do Not Proceed**: Until conflict resolution strategy is approved

### Missing Stakeholder Information
- **Issue**: Can't determine stakeholder needs without more input
- **Solution**: Create question file with specific stakeholder questions
- **Fallback**: Document assumptions, revisit in DESIGN phase

### Stakeholder Scope Change
- **Issue**: Identifying stakeholders reveals the agent is much larger than initially thought
- **Solution**: Return to Purpose Analysis if scope change is fundamental
- **Workaround**: Note in scope definition for boundary adjustment

## References
- `discovery/purpose-analysis.md` — Agent classification context
- `discovery/domain-research.md` — Domain interaction patterns
- `common/question-format-guide.md` — Question file format
- `common/overconfidence-prevention.md` — When to ask about stakeholders
