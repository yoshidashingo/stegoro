# PRIORITY: This workflow OVERRIDES all other built-in workflows
# When user requests steering policy creation, ALWAYS follow this workflow FIRST

## Adaptive Workflow Principle
**The workflow adapts to the target agent's needs, not the other way around.**

The AI model intelligently assesses what is needed based on:
1. User's stated purpose and target agent description
2. Target agent's domain complexity and risk profile
3. Number and types of stakeholders involved
4. Desired quality level and scope of the policy set

## MANDATORY: Rule Details Loading
**CRITICAL**: When performing any phase, you MUST read and use relevant content from rule detail files in `rule-details/` directory.

**Common Rules**: ALWAYS load all 11 common rules at workflow start:
- Load `common/process-overview.md` for workflow overview
- Load `common/session-continuity.md` for session resumption guidance
- Load `common/content-validation.md` for content validation requirements
- Load `common/question-format-guide.md` for question formatting rules
- Load `common/error-handling.md` for recovery procedures
- Load `common/overconfidence-prevention.md` for uncertainty handling
- Load `common/terminology.md` for standardized terminology
- Load `common/quality-standards.md` for 15-dimension quality benchmark
- Load `common/output-structure-patterns.md` for artifact structure patterns
- Load `common/implementation-knowhow.md` for domain-specific implementation patterns
- Reference these throughout the workflow execution

## MANDATORY: Working Directory Isolation
**CRITICAL**: Each steering policy creation MUST use an isolated working directory to prevent data mixing across multiple runs.

**Working Directory**: `steering-docs/<agent-name>/`

- `<agent-name>` is determined during Purpose Analysis and used throughout the workflow
- All working artifacts (state files, question files, plans, reports) go under `steering-docs/<agent-name>/`
- State file: `steering-docs/<agent-name>/steering-state.md`
- Audit file: `steering-docs/<agent-name>/audit.md`
- Phase artifacts: `steering-docs/<agent-name>/<phase-name>/`
- This isolation ensures multiple concurrent or sequential policy generations do not interfere with each other

**Agent Name Convention**: `<agent-name>` must be lowercase, hyphen-separated, ASCII-only (e.g., `code-reviewer`, `ci-pipeline-manager`). No spaces, slashes, or special characters.

**Initialization**: At the start of Purpose Analysis, create the working directory:
```bash
mkdir -p steering-docs/<agent-name>/discovery
```

## MANDATORY: Content Validation
**CRITICAL**: Before creating ANY file, you MUST validate content according to `common/content-validation.md` rules:
- Validate Mermaid diagram syntax
- Validate ASCII art diagrams
- Escape special characters properly
- Provide text alternatives for complex visual content
- Test content parsing compatibility

## MANDATORY: Question File Format
**CRITICAL**: When asking questions at any phase, you MUST follow question format guidelines.

**See `common/question-format-guide.md` for complete question formatting rules including**:
- Multiple choice format (A, B, C, D, E options)
- [Answer]: tag usage
- Answer validation and ambiguity resolution

## MANDATORY: Custom Welcome Message
**CRITICAL**: When starting ANY steering policy creation request, you MUST display the welcome message.

**How to Display Welcome Message**:
1. Load the welcome message from `rule-details/common/welcome-message.md`
2. Display the complete message to the user
3. This should only be done ONCE at the start of a new workflow
4. Do NOT load this file in subsequent interactions to save context space

# Steering Policy Maker — Adaptive Workflow

---

# DISCOVERY PHASE

**Purpose**: Understanding WHAT steering policy to create and WHY

**Focus**: Analyze the user's purpose, research the domain, identify stakeholders, and define scope

**Stages in DISCOVERY PHASE**:
- Purpose Analysis (ALWAYS)
- Domain Research (ALWAYS — Adaptive depth)
- Stakeholder Identification (CONDITIONAL)
- Scope Definition (ALWAYS)

---

## Purpose Analysis (ALWAYS EXECUTE)

1. **MANDATORY**: Determine target agent name from user's request. Create working directory: `mkdir -p steering-docs/<agent-name>/discovery`
2. **MANDATORY**: Log initial user request in audit.md with complete raw input
3. Load all steps from `discovery/purpose-analysis.md`
3. Execute purpose analysis:
   - Analyze user's stated purpose and intent
   - Classify target agent type using **4-type analysis templates** (Process / Task / Analytical / Hybrid)
   - Identify core capabilities the agent needs
   - Determine complexity level (Simple / Standard / Complex) using **quantitative indicators** (phase count, stage count, quality dimension count)
   - Map purpose to known agent patterns
4. **MANDATORY**: Create purpose analysis question file if clarification needed
5. **Wait for Explicit Approval**: Present purpose analysis summary — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log findings and user response in audit.md
7. Present completion message to user (see purpose-analysis.md for message formats)

## Domain Research (ALWAYS EXECUTE — Adaptive Depth)

**Always executes** but depth varies based on domain familiarity and complexity:
- **Minimal**: Well-known domain with clear best practices — minimum 3 research items
- **Standard**: Domain with some nuance requiring research — minimum 7 research items
- **Comprehensive**: Novel or high-risk domain requiring deep research — minimum 12 research items

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `discovery/domain-research.md`
3. Execute domain research with **MCP tool integration**:
   - Context7 → official documentation lookup
   - Exa → web research for best practices and patterns
   - gh search → existing implementation examples
   - **Fallback**: If MCP tools unavailable, ask user directly for domain knowledge
4. Research domain-specific best practices, pitfalls, quality standards, terminology, and reference implementations
5. Execute at appropriate depth (minimal/standard/comprehensive) meeting minimum item counts
6. **Wait for Explicit Approval**: Present domain research summary — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Stakeholder Identification (CONDITIONAL)

**Execute IF**:
- Multiple user types or personas will interact with the target agent
- Different stakeholders have different needs or perspectives
- The target agent serves cross-functional teams
- Access control or role-based behavior is needed

**Skip IF**:
- Single user type with clear, unified needs
- Simple task agent with one interaction pattern
- User explicitly confirms single stakeholder

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `discovery/stakeholder-identification.md`
3. Execute stakeholder identification:
   - Identify all user types and personas
   - Map stakeholder needs and priorities
   - Define interaction patterns per stakeholder
   - Identify conflicting requirements between stakeholders
4. **Wait for Explicit Approval**: Present stakeholder map — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

## Scope Definition (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `discovery/scope-definition.md`
3. Load all prior DISCOVERY context:
   - Purpose analysis results
   - Domain research findings
   - Stakeholder identification (if executed)
4. Execute scope definition:
   - Define policy set boundaries (what's in scope, what's out)
   - Estimate number of phases and stages for target agent (5 phases including PACKAGING)
   - **Measure existing files** with `wc -l` for accurate baseline (brush-up only)
   - Estimate number of rule files needed with **target line count differentials**
   - Define quality level target (Standard / Premium)
   - Create preliminary directory structure including **plugin structure**
   - Create **migration plan** template (brush-up only)
5. **Wait for Explicit Approval**: Present scope definition with estimated effort — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

---

# DESIGN PHASE

**Purpose**: Architecting the steering policy structure

**Focus**: Design the workflow, rules, and quality mechanisms for the target agent

**Stages in DESIGN PHASE**:
- Workflow Architecture (ALWAYS)
- Common Rules Design (ALWAYS)
- Phase Rules Design (ALWAYS)
- Quality Mechanisms Design (ALWAYS)

---

## Workflow Architecture (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `design/workflow-architecture.md`
3. **MANDATORY**: Load content validation rules from `common/content-validation.md`
4. Load all prior context:
   - Purpose analysis (agent type classification)
   - Domain research (best practices, standards)
   - Stakeholder identification (if executed)
   - Scope definition (boundaries, estimates)
5. Execute workflow architecture design:
   - Design phase structure based on agent type (including PACKAGING phase for Complex agents)
   - Define stages within each phase with ALWAYS/CONDITIONAL classification
   - Design stage dependencies and flow
   - Create workflow visualization (VALIDATE Mermaid syntax before writing)
   - Define checkpoint and approval gate placement
   - Design state tracking mechanism
   - **Design repair judgment tree** — quality dimension → problem classification → return target mapping
   - **Design repair loop control** — max retries, same-target limits, escalation conditions
6. **MANDATORY**: Validate all content before file creation per content-validation.md rules
7. **Wait for Explicit Approval**: Present workflow architecture with visualization — DO NOT PROCEED until user confirms
8. **MANDATORY**: Log user's response in audit.md with complete raw input

## Common Rules Design (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `design/common-rules-design.md`
3. Load all prior context including workflow architecture
4. Execute common rules design:
   - Select which standard common rules apply (from template set)
   - Identify domain-specific common rules needed
   - Design welcome message content
   - Design question format adaptations
   - Design content validation rules specific to domain
   - Design error handling procedures
   - Design session continuity approach
   - **Design plugin-related rules** (if PACKAGING phase is included)
5. **Wait for Explicit Approval**: Present common rules design — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

## Phase Rules Design (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `design/phase-rules-design.md`
3. Load all prior context including workflow architecture and common rules design
4. Execute phase rules design:
   - For each phase in the target workflow:
     - Define stage-level rule files needed
     - Design each rule file's structure and content outline
     - Define inter-stage dependencies
     - Design stage completion criteria
     - Design approval gate content
     - **Identify domain content injection points** for each stage rule file
   - **Design PACKAGING phase rules** (P1/P2/P3) if applicable
   - Map rule files to directory structure
   - Verify complete coverage of all workflow paths
5. **Wait for Explicit Approval**: Present phase rules design with file list — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

## Quality Mechanisms Design (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `design/quality-mechanisms-design.md`
3. Load all prior context including workflow architecture and rules designs
4. Execute quality mechanisms design:
   - Design checkpoint system (plan-level and stage-level)
   - Design audit trail format and requirements
   - Design content validation rules
   - Design overconfidence prevention measures
   - Design error handling and recovery procedures
   - Design completeness verification approach
   - Map to **15 quality dimensions** (11 standard + 4 domain-specific)
   - **Design repair judgment tree** with dimension → classification → target mapping
   - **Design 3-layer automated testing** (structure + content + smoke)
   - **Design loop control rules** (max 3 retries, same-target 2x escalation)
5. **Wait for Explicit Approval**: Present quality mechanisms design — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

---

# GENERATION PHASE

**Purpose**: Creating the actual steering policy files

**Focus**: Generate all rule files with full content, following the approved designs

**Stages in GENERATION PHASE**:
- Core Workflow Generation (ALWAYS)
- Common Rules Generation (ALWAYS)
- Phase Rules Generation (ALWAYS)
- Integration Validation (ALWAYS)

---

## Core Workflow Generation (ALWAYS EXECUTE)

**Core Workflow Generation has two parts within one stage**:
1. **Part 1 - Planning**: Create detailed generation plan with explicit steps
2. **Part 2 - Generation**: Execute approved plan to generate the core workflow file

**Adaptive Depth** — core-workflow output varies by complexity:
- **Light** (Simple/Task Agent): 200-300 lines, 2-3 phases, stage overviews
- **Medium** (Standard Agent): 300-450 lines, 3-4 phases, execution summaries + approval gates
- **Heavy** (Complex/Process Agent): 450-600 lines, 4-5 phases, detailed steps + adaptive depth + error handling

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `generation/core-workflow-generation.md`
3. Load all DESIGN phase artifacts:
   - Workflow architecture
   - Common rules design
   - Phase rules design
   - Quality mechanisms design
4. **PART 1 - Planning**: Create core workflow generation plan with checkboxes, get user approval
5. **PART 2 - Generation**: Execute approved plan to generate the target's core-workflow.md
6. **MANDATORY**: Present standardized 2-option completion message — DO NOT use emergent behavior
7. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
8. **MANDATORY**: Log user's response in audit.md with complete raw input

## Common Rules Generation (ALWAYS EXECUTE)

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `generation/common-rules-generation.md`
3. Load all DESIGN phase artifacts and generated core workflow
4. Execute common rules generation using **Light/Medium/Heavy templates**:
   - Generate each common rule file following the design specifications
   - Apply domain-specific adaptations
   - **Follow generation order**: terminology → quality-standards → remaining files
   - Cross-reference with core workflow for consistency
   - Validate content before writing each file
   - **Check line count guidelines** for each file
5. **MANDATORY**: Present standardized 2-option completion message
6. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Phase Rules Generation (ALWAYS EXECUTE)

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `generation/phase-rules-generation.md`
3. Load all DESIGN phase artifacts, core workflow, and common rules
4. Execute phase rules generation with **domain content injection**:
   - For each phase in the target workflow:
     - Generate each stage rule file following the design
     - **Execute domain content injection heuristics** (5 steps):
       - 3c-1: Load Domain Research Summary
       - 3c-2: Map domain content to Execution Steps, Examples, Error Handling
       - 3c-3: Measure domain specificity rate per file (terminology terms ÷ total lines)
       - 3c-4: Flag files below 40% for additional injection
       - 3c-5: Inject additional examples, procedures, checks into flagged files
     - Ensure stage rule files reference appropriate common rules
     - Insert **minimum 2 GOOD/BAD examples** per file
     - Validate inter-stage references and dependencies
     - Validate content before writing each file
   - **Generate PACKAGING phase rules** (P1/P2/P3) if applicable
5. **MANDATORY**: Present standardized 2-option completion message
6. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Integration Validation (ALWAYS EXECUTE)

**3-Layer Testing** — Structure (file existence, references, Markdown, flow paths) + Content (Dim 12-15 pre-validation) + Smoke (4 agent types traversal). All layers must PASS with 0 errors.

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `generation/integration-validation.md`
3. Load ALL generated files and execute 3-layer testing
4. Generate validation report (see `generation/integration-validation.md` for report format)
5. **If validation FAIL**: Route by test layer — Structure FAIL → G1, Content FAIL (Dim 12-15) → G3, Smoke FAIL → G1 (flow break)
6. **Wait for Explicit Approval**: Present validation report — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

---

# REFINEMENT PHASE

**Purpose**: Ensuring quality in the generated policy set

**Focus**: Review, validate, and calibrate the complete policy set against 15 quality dimensions

**Stages in REFINEMENT PHASE**:
- Completeness Review (ALWAYS)
- Consistency Review (ALWAYS)
- Quality Calibration (ALWAYS)

---

## Completeness Review (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `refinement/completeness-review.md`
3. Load ALL generated policy files
4. Execute completeness review — **two axes**:
   - **Structural completeness**: All workflow paths have corresponding rule files, all ALWAYS/CONDITIONAL classifications documented, all approval gates defined, all error scenarios have handling procedures
   - **Content depth**: All Phase Rule files have Purpose/Prerequisites/Steps/Error Handling/Completion Message sections, all files have >=2 GOOD/BAD examples, all GENERATION files have artifact templates, all Error Handling sections contain domain-specific scenarios
5. **If gaps found**: Route via repair judgment tree:
   - File missing / flow break → `[REPAIR: structure]` → G1
   - Content insufficient (no templates, no examples) → `[REPAIR: content]` → G3
6. **Wait for Explicit Approval**: Present completeness report — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Consistency Review (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `refinement/consistency-review.md`
3. Load ALL generated policy files
4. Execute consistency review:
   - Verify terminology usage matches terminology.md across all files (including Common Misuse detection)
   - Verify structural patterns are consistent (headings, sections, formatting)
   - Verify completion message formats are consistent
   - Verify question file formats follow the guide
   - Verify error handling patterns are consistent
   - **Measure domain specificity rate** per Phase Rule file (Dim 12 measurement)
   - **Verify 4-agent-type consistency** — references to Process/Task/Analytical/Hybrid are consistent
   - Generate consistency report with findings and **domain specificity heatmap**
5. **If inconsistencies found**: Fix minor issues in-place (terminology corrections, formatting). Structural issues are measured and reported to R3 Quality Calibration for repair routing — R2 does NOT trigger repair loops.
6. **Wait for Explicit Approval**: Present consistency report — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Quality Calibration (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `refinement/quality-calibration.md`
3. Load ALL generated policy files AND quality standards reference
4. Execute quality calibration against **15 dimensions** (Dim 1-11: AI-DLC standard, Dim 12-15: domain-specific — see `common/quality-standards.md` for full definitions):
   - Dim 1-11: Adaptive Workflow, Mandatory Checkpoints, Question File Format, Content Validation, Audit Trail, Error Handling, Overconfidence Prevention, Depth Levels, Session Continuity, Terminology Standardization, Standardized Completion Messages
   - Dim 12: Domain Specificity Rate >= 40%
   - Dim 13: Example Coverage >= 2/file
   - Dim 14: Artifact Template Completeness = 100%
   - Dim 15: Pitfall Reference Rate >= 50%
5. Score each dimension (Pass / Partial / Fail) with **evidence** (file path:line number, count values)
6. **If any dimension FAIL**: Apply repair judgment tree (see below)
7. **Approval criteria**:
   - **APPROVED**: 15/15 PASS → proceed to PACKAGING
   - **CONDITIONAL APPROVAL**: 13+ PASS, FAIL only in Dim 12-15 → user decides
   - **NEEDS REMEDIATION**: <=12 PASS or Dim 1-11 FAIL → repair loop
8. **Wait for Explicit Approval**: Present calibration scorecard — DO NOT PROCEED until user confirms
9. **MANDATORY**: Log user's response in audit.md with complete raw input

---

# PACKAGING PHASE

**Purpose**: Packaging validated policies into Claude Code plugin format

**Focus**: Plugin structure generation, automated validation, and migration

**Stages in PACKAGING PHASE**:
- Plugin Structure Generation (ALWAYS)
- Automated Validation (ALWAYS)
- Migration Execution (CONDITIONAL)

---

## Plugin Structure Generation (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `packaging/plugin-structure-generation.md`
3. Load REFINEMENT results (APPROVED: 15/15 PASS, or CONDITIONAL APPROVAL: 13+ PASS with user-approved progression) and Scope Definition
4. Execute plugin structure generation:
   - Generate plugin configuration: plugin.json + marketplace.json
   - Generate agent definitions (orchestrator + specialist agents)
   - Generate skill entry points (SKILL.md files, 75-150 lines each)
   - Generate commands (new-policy, resume-policy, validate-policy)
   - Generate rules file (standards enforcement)
   - Copy improved rule-details files into plugin structure
   - Validate against `common/implementation-knowhow.md` skill design patterns
5. **MANDATORY**: Present standardized 2-option completion message
6. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Automated Validation (ALWAYS EXECUTE)

**3-Layer Testing (Plugin Version)** — Structure (plugin.json fields, file existence) + Content (line count ranges: SKILL.md 75-150, core-workflow 200-600, Phase Rule 100-300, Common Rule 50-200) + Smoke (4 agent types flow traversal). All must PASS.

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `packaging/automated-validation.md`
3. Load plugin structure from P1 and execute 3-layer testing
4. **If FAIL**: Return to P1 for targeted fixes (max 2 P2→P1 loops)
5. **Wait for Explicit Approval**: Present validation report — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

## Migration Execution (CONDITIONAL)

**Execute IF**:
- Existing policy brush-up (legacy structure exists, e.g., `.steering/` directory)

**Skip IF**:
- New policy generation (no migration source)

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `packaging/migration-execution.md`
3. Execute migration:
   - Identify migration source directory
   - Create file path mapping (old → new)
   - Copy improved files to plugin structure
   - Verify integrity post-copy
   - Update all references (CLAUDE.md, routing rules, etc.)
   - Mark legacy directory as deprecated (do NOT delete)
4. **Wait for Explicit Approval**: Present migration report — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

---

## Key Principles

- **Adaptive Execution**: Only execute stages that add value for the specific target agent
- **Transparent Planning**: Always show execution plan before starting
- **User Control**: User can request stage inclusion/exclusion
- **Progress Tracking**: Update steering-state.md with executed and skipped stages
- **Complete Audit Trail**: Log ALL user inputs and AI responses in audit.md with timestamps
  - **CRITICAL**: Capture user's COMPLETE RAW INPUT exactly as provided
  - **CRITICAL**: Never summarize or paraphrase user input in audit log
  - **CRITICAL**: Log every interaction, not just approvals
- **Quality Focus**: Complex agents get full treatment, simple agents stay efficient
- **Content Validation**: Always validate content before file creation per content-validation.md rules
- **NO EMERGENT BEHAVIOR**: All phases MUST use standardized 2-option completion messages as defined in their respective rule files. DO NOT create 3-option menus or other emergent navigation patterns.
- **Repair Loop Discipline**: Follow the repair judgment tree for all FAIL/gap routing. Max 3 repair loops total. Escalate to user on same-target 2nd return.

## MANDATORY: Plan-Level Checkbox Enforcement

### MANDATORY RULES FOR PLAN EXECUTION
1. **NEVER complete any work without updating plan checkboxes**
2. **IMMEDIATELY after completing ANY step described in a plan file, mark that step [x]**
3. **This must happen in the SAME interaction where the work is completed**
4. **NO EXCEPTIONS**: Every plan step completion MUST be tracked with checkbox updates

### Two-Level Checkbox Tracking System
- **Plan-Level**: Track detailed execution progress within each stage
- **Stage-Level**: Track overall workflow progress in steering-state.md
- **Update immediately**: All progress updates in SAME interaction where work is completed

## Prompts Logging Requirements
- **MANDATORY**: Log EVERY user input (prompts, questions, responses) with timestamp in audit.md
- **MANDATORY**: Capture user's COMPLETE RAW INPUT exactly as provided (never summarize)
- **MANDATORY**: Log every approval prompt with timestamp before asking the user
- **MANDATORY**: Record every user response with timestamp after receiving it
- **CRITICAL**: ALWAYS append changes to EDIT audit.md file, NEVER use tools and commands that completely overwrite its contents
- Use ISO 8601 format for timestamps (YYYY-MM-DDTHH:MM:SSZ)
- Include stage context for each entry

### Audit Log Format:
```markdown
## [Stage Name or Interaction Type]
**Timestamp**: [ISO timestamp]
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[AI's response or action taken]"
**Context**: [Stage, action, or decision made]

---
```

### Additional Log Types

- **Approval Log** (each CP): Add `**Checkpoint**: CP-[N]` and `**Result**: APPROVED/SKIPPED`
- **Repair Loop Log**: Add `**Loop #**: [N]` `**Source→Target**: [Stage→Stage]` `**Dimension**: Dim [N]`
- **Validation Log** (G4/P2): Add `**Test Layer**: [structure/content/smoke]` `**Result**: PASS/FAIL`
- **Escalation Log**: Add `**Loop Count**: [N]` `**User Choice**: [continue/abort/rescope]`

### Correct Tool Usage for audit.md

CORRECT:

1. Read the audit.md file
2. Append/Edit the file to make changes

WRONG:

1. Read the audit.md file
2. Completely overwrite the audit.md with the contents of what you read, plus the new changes you want to add to it

---

## Repair Judgment Tree

When a quality dimension FAIL or completeness gap is detected, route to the appropriate stage.

**Full 15-dimension mapping**: See `design/quality-mechanisms-design.md` Section 9 for the complete dimension→classification→target table.

### Routing Summary

```
FAIL / Gap detected
├── Structural (file missing, reference broken, flow break)
│   → G1 (Core Workflow Generation)
├── Content (domain specificity low, examples missing, templates missing)
│   → G3 (Phase Rules Generation) or G2 (Common Rules)
├── Design (phase structure inappropriate, stage classification error)
│   → E1 (Workflow Architecture)
└── Criteria (dimension definition deficiency)
    → E4 (Quality Mechanisms Design)
```

### Repair Loop Control

- **Max retries**: 3 total across entire workflow
- **Same-target limit**: 2nd return to same stage triggers user escalation
- **Escalation**: "Repair loop #{N}. Choose: A) Continue, B) Abort (with quality note), C) Rescope"
- **P2 loop**: P2→P1 max 2 times (separate counter)
- **History**: Record all loops in steering-state.md Repair Loop History

---

## Domain Adaptation — Agent Type Classification

The Steering Policy Maker classifies target agents into four types, which determine the generated policy structure:

| Type | Example | Pattern | Typical Phases |
|------|---------|---------|---------------|
| **Process** | AI-DLC, CI/CD Pipeline | Plan → Approve → Execute → Verify | 3-5 phases, 3-7 stages each |
| **Task** | Code Reviewer, Bug Triager | Analyze → Process → Report | 2-3 phases, 2-4 stages each |
| **Analytical** | Document Generator, Data Analyzer | Ingest → Transform → Produce | 2-4 phases, 2-5 stages each |
| **Hybrid** | DevOps Assistant, Project Manager | Context-dependent switching | 3-6 phases, variable stages |

## Generated Output Directory Structure

### Policy Files (Standard)

```text
.<agent-name>/
├── <agent-name>-rules/
│   └── core-workflow.md                    Master orchestrator
└── <agent-name>-rule-details/
    ├── common/                             Cross-phase rules (11 files)
    └── <phase-N>/                          Phase-specific rules per stage
```

### Plugin Structure (PACKAGING phase output)

```text
<agent-name>/
├── .claude-plugin/                         plugin.json + marketplace.json
├── agents/                                 Specialist agent definitions
├── skills/                                 Skill entry points (SKILL.md)
├── commands/                               Command definitions
├── rules/                                  Quality standards rules
└── <agent-name>-rule-details/              Improved rule files (copied from policy)
```

**CRITICAL RULES**: Generated policy files go inside `.<agent-name>/` directory. Each target agent gets its own dot-directory. All file references in core-workflow.md must resolve to actual files. Plugin structure is generated only in PACKAGING phase. Line limits: SKILL.md 75-150, core-workflow 200-600.
