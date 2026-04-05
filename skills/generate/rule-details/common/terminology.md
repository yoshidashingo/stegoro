# Steering Policy Maker — Terminology Glossary

## Core Terminology

| Term | Definition | Common Misuse | Related Terms |
|------|-----------|---------------|---------------|
| Steering Policy | A structured set of markdown-based rules and instructions that guide an AI agent through a specific workflow. Defines WHAT the agent does, WHEN it does it, and HOW it ensures quality. | Confused with "prompt" or "system instruction" — a steering policy is a multi-file structured system, not a single prompt | Steering Policy Set, Core Workflow |
| Steering Policy Set | The complete collection of files (core workflow + common rules + phase rules) that together form a comprehensive steering policy for a target agent. | Using "policy" when meaning "policy set" — a single file is not a policy set | Policy Files, Core Workflow |
| Target Agent | The AI agent for which steering policies are being created. The target agent will ultimately use the generated policy set to guide its behavior. | Confused with "SPM" — SPM creates policies, the target agent uses them | Agent Type Classification |
| Steering Policy Maker (SPM) | This system — the meta-agent that creates steering policies for other agents. Guided by the files in `rule-details/`. | Referring to SPM as "the agent" when discussing the target agent | Meta-Agent |

---

## Phase vs Stage

**Phase**: One of the five high-level lifecycle phases in the Steering Policy Maker

| Phase | Purpose | Focus |
|-------|---------|-------|
| DISCOVERY | Understanding and Research | WHAT and WHY |
| DESIGN | Architecture and Planning | HOW to structure |
| GENERATION | File Creation | BUILD the policies |
| REFINEMENT | Quality Assurance | VERIFY and CALIBRATE |
| PACKAGING | Plugin Packaging | DELIVER as skill/plugin |

**Stage**: An individual workflow activity within a phase
- Each stage has specific prerequisites, steps, and outputs
- Stages are classified as ALWAYS or CONDITIONAL

**Common Misuse**: Using "phase" when referring to a stage (e.g., "the Purpose Analysis phase" — should be "stage"). Using "stage" when referring to a phase (e.g., "the DISCOVERY stage" — should be "phase").

---

## Five-Phase Lifecycle

### Phase 1: DISCOVERY
**Stages**: Purpose Analysis (ALWAYS), Domain Research (ALWAYS — Adaptive Depth), Stakeholder Identification (CONDITIONAL), Scope Definition (ALWAYS)
**Outputs**: Purpose classification, domain best practices, stakeholder map, scope definition
**Location**: `discovery/` directory

### Phase 2: DESIGN
**Stages**: Workflow Architecture (ALWAYS), Common Rules Design (ALWAYS), Phase Rules Design (ALWAYS), Quality Mechanisms Design (ALWAYS)
**Outputs**: Workflow diagram, rules inventory, file list, quality plan
**Location**: `design/` directory

### Phase 3: GENERATION
**Stages**: Core Workflow Generation (ALWAYS), Common Rules Generation (ALWAYS), Phase Rules Generation (ALWAYS), Integration Validation (ALWAYS)
**Outputs**: Complete set of policy files, validation report
**Location**: `generation/` directory

### Phase 4: REFINEMENT
**Stages**: Completeness Review (ALWAYS), Consistency Review (ALWAYS), Quality Calibration (ALWAYS)
**Outputs**: Completeness report, consistency report, calibration scorecard
**Location**: `refinement/` directory

### Phase 5: PACKAGING
**Stages**: Plugin Structure Generation (ALWAYS), Automated Validation (ALWAYS), Migration Execution (CONDITIONAL)
**Outputs**: Plugin files (plugin.json, SKILL.md, agents/, commands/), validation report, migration report
**Location**: `packaging/` directory

---

## Agent Type Classification

| Term | Definition | Common Misuse | Related Terms |
|------|-----------|---------------|---------------|
| Process Agent | An agent guiding users through a multi-phase workflow with sequential stages, checkpoints, and state tracking. Most complex type. | Assuming all agents are Process Agents — simpler types exist | Plan → Approve → Execute → Verify |
| Task Agent | An agent focused on executing a specific, bounded task with preparation, execution, and reporting. | Calling a multi-phase workflow a "task" — Task Agents have 2-3 phases max | Analyze → Process → Report |
| Analytical Agent | An agent that processes inputs, applies analysis or transformation, and produces structured outputs. | Confusing with Task Agent — Analytical Agents are data-driven and template-based | Ingest → Transform → Produce |
| Hybrid Agent | An agent combining characteristics of multiple types, switching between modes based on context. | Defaulting to Hybrid when unsure — verify specific characteristics first | Context-dependent switching |

---

## Execution Classification

| Term | Definition | Common Misuse | Related Terms |
|------|-----------|---------------|---------------|
| ALWAYS Execute | A stage that MUST execute regardless of context. Foundational to the workflow. | Marking a stage ALWAYS when it could be CONDITIONAL — evaluate skip criteria | CONDITIONAL, Adaptive Depth |
| CONDITIONAL Execute | A stage that executes only when specific conditions are met. Has explicit Execute IF / Skip IF criteria. | Omitting Skip IF criteria — both conditions must be documented | Execute IF, Skip IF |
| Adaptive Depth | A stage that ALWAYS executes but adjusts depth (Minimal/Standard/Comprehensive) based on complexity. | Confusing with CONDITIONAL — Adaptive Depth stages always run, just at varying detail | Minimal, Standard, Comprehensive |

---

## Quality Dimensions (15 Dimensions)

### Standard Dimensions (Dim 1-11, AI-DLC)

| # | Dimension | Definition | Common Misuse |
|---|-----------|-----------|---------------|
| 1 | Adaptive Workflow | Stages classified as ALWAYS/CONDITIONAL with explicit conditions | Marking all stages ALWAYS without evaluating |
| 2 | Mandatory Checkpoints | Critical decisions require explicit user approval | Skipping approval for "obvious" decisions |
| 3 | Question File Format | All questions use multi-choice + "Other" + [Answer]: tags | Asking questions in chat instead of question files |
| 4 | Content Validation | Generated content validated before file creation | Writing files without pre-validation |
| 5 | Audit Trail | Complete logging with ISO 8601 timestamps, raw user inputs | Summarizing user input instead of capturing verbatim |
| 6 | Error Handling | Phase-specific recovery procedures for all severity levels | Generic "retry" without phase-specific guidance |
| 7 | Overconfidence Prevention | "When in doubt, ask" — default to asking rather than assuming | Assuming domain knowledge without verification |
| 8 | Depth Levels | Adaptive detail based on complexity | Applying same depth to all agent types |
| 9 | Session Continuity | State tracking via state file, session resumption | Restarting from scratch on session resume |
| 10 | Terminology Standardization | Consistent terminology enforced via glossary | Using informal synonyms instead of defined terms |
| 11 | Standardized Completion Messages | Every stage ends with REVIEW REQUIRED + WHAT'S NEXT (2 options) | Creating 3+ option menus or emergent navigation |

### Domain-Specific Dimensions (Dim 12-15, SPM)

| # | Dimension | Definition | Measurement | Threshold | Common Misuse |
|---|-----------|-----------|-------------|-----------|---------------|
| 12 | Domain Specificity Rate | Ratio of domain-specific lines to total lines in Phase Rule files | Domain-specific lines ÷ (total - blank - heading lines) | >= 40% | Counting domain word frequency instead of line-level assessment |
| 13 | Example Coverage | Number of GOOD/BAD examples per Phase Rule file | Count example code blocks per file | >= 2/file | Counting generic examples that lack domain context |
| 14 | Artifact Template Completeness | Percentage of output-producing files that include templates | Files with templates ÷ files that should have templates | = 100% | Confusing with "having a section header" — actual template content required |
| 15 | Pitfall Reference Rate | Ratio of error handling items referencing domain pitfalls | Error items citing domain-research pitfalls ÷ total error items | >= 50% | Citing generic errors instead of domain-research-derived pitfalls |

---

## Repair and Loop Control

| Term | Definition | Common Misuse | Related Terms |
|------|-----------|---------------|---------------|
| Repair Loop | A quality-dimension-driven cycle that returns to a prior stage to fix a specific FAIL. Triggered by Quality Calibration FAIL or Completeness Review gap. | Treating as "debug" or "retry" — repair loops follow the repair judgment tree, not ad-hoc fixes | Repair Judgment Tree, Escalation |
| Repair Judgment Tree | A mapping from quality dimension FAIL → problem classification → return target stage. Determines where to route a repair. | Manually choosing a return target without consulting the tree | Dim→Classification→Target |
| Escalation | User intervention triggered when repair loop limits are reached (max 3 total, or same target 2x). User chooses: continue, abort, or rescope. | Silently retrying instead of escalating | Max Retries, Same-Target Limit |
| Max Retries | Maximum number of repair loops allowed (3 total across workflow). | Counting per-phase instead of across the entire workflow | Loop Control |
| Same-Target Limit | Warning triggered on 2nd return to the same stage. Suggests the problem classification itself may be wrong. | Ignoring repeated returns to the same stage | Escalation |

---

## PACKAGING Terminology

| Term | Definition | Common Misuse | Related Terms |
|------|-----------|---------------|---------------|
| Plugin Structure | The Claude Code plugin directory layout (.claude-plugin/, agents/, skills/, commands/, rules/). | Confusing with "policy set" — plugin structure is the delivery format, not the policy content | plugin.json, marketplace.json |
| SKILL.md | The skill entry point file (75-150 lines) that defines activation triggers, core rules, and quality gates. | Writing full implementation in SKILL.md — it should be concise and delegate to rule-details | Skills, Activation Trigger |
| plugin.json | Plugin metadata file defining name, version, agents, skills, commands, and rules references. | Including rule-details file paths directly — plugin.json references top-level entry points only | marketplace.json |
| marketplace.json | Marketplace publication metadata (display name, description, tags, author). | Confusing with plugin.json — marketplace.json is for discovery/display, not execution | Plugin Structure |
| Migration | The process of moving improved policy files from a legacy structure to the new plugin structure. | Deleting legacy files — migration marks them as deprecated, does not delete | P3: Migration Execution |
| Smoke Test | A lightweight end-to-end flow traversal verifying all stages are reachable for all 4 agent types. | Confusing with full integration testing — smoke tests check reachability, not content quality | Automated Validation, 3-Layer Testing |
| Validation Agent | An agent or command that executes the 3-layer test suite (structure + content + smoke). | Confusing with Quality Calibration — validation is automated, calibration is AI-assessed | P2: Automated Validation |

---

## Artifact Types

| Type | Description | Examples |
|------|-------------|---------|
| Policy Files | Markdown files constituting the steering policy set | core-workflow.md, common/*.md, phase/*.md |
| Working Artifacts | Files created during the policy creation process | question files, plans, reports |
| State Files | Files tracking workflow progress | steering-state.md, audit.md |
| Plugin Files | Files for Claude Code plugin packaging | plugin.json, SKILL.md, agents/*.md, commands/*.md |

---

## Common Abbreviations

- **SPM**: Steering Policy Maker
- **AI-DLC**: AI-Driven Development Life Cycle (reference implementation)
- **CP**: Checkpoint (e.g., CP-01 through CP-18)
- **Dim**: Dimension (e.g., Dim 12 = Domain Specificity Rate)

---

## Terminology Guidelines

### When Referring to This System
- "Steering Policy Maker" or "SPM" — the system creating policies
- "steering policy creation workflow" — the process this system follows

### When Referring to Target Agents
- "target agent" — the agent being created
- "target workflow" — the workflow designed for the target agent
- "generated policy set" — the output files for the target agent

### When Referring to Quality
- "15 quality dimensions" — the complete quality criteria (11 AI-DLC + 4 SPM)
- "AI-DLC-level quality" — the quality standard the generated policies should achieve
- "Premium quality" — all 15 dimensions PASS
