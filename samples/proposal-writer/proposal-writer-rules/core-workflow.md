# PRIORITY: This workflow OVERRIDES all other built-in workflows
# When user provides an RFP or requests proposal creation, ALWAYS follow this workflow FIRST

## Adaptive Workflow Principle
**The workflow adapts to the RFP complexity, not the other way around.**

The AI model intelligently assesses what stages are needed based on:
1. RFP document size, structure, and clarity
2. Required response components (technical, management, pricing)
3. Evaluation criteria complexity and weighting
4. Competitive context (sole source vs. competitive bid)
5. Compliance and regulatory requirements

## MANDATORY: Rule Details Loading
**CRITICAL**: When performing any phase, you MUST read and use relevant content from rule detail files in `proposal-writer-rule-details/` directory.

**Common Rules**: ALWAYS load common rules at workflow start:
- Load `common/process-overview.md` for workflow overview
- Load `common/session-continuity.md` for session resumption guidance
- Load `common/content-validation.md` for content validation requirements
- Load `common/question-format-guide.md` for question formatting rules
- Reference these throughout the workflow execution

## MANDATORY: Content Validation
**CRITICAL**: Before creating ANY file, you MUST validate content according to `common/content-validation.md` rules:
- Validate document structure and formatting
- Escape special characters properly
- Verify cross-references between proposal sections
- Ensure compliance matrix entries map correctly to RFP requirements

## MANDATORY: Question File Format
**CRITICAL**: When asking questions at any phase, you MUST follow question format guidelines.

**See `common/question-format-guide.md` for complete question formatting rules including**:
- Multiple choice format (A, B, C, D, E options)
- [Answer]: tag usage
- Answer validation and ambiguity resolution

## MANDATORY: Custom Welcome Message
**CRITICAL**: When starting ANY proposal creation workflow, you MUST display the welcome message.

**How to Display Welcome Message**:
1. Load the welcome message from `proposal-writer-rule-details/common/welcome-message.md`
2. Display the complete message to the user
3. This should only be done ONCE at the start of a new workflow
4. Do NOT load this file in subsequent interactions to save context space

# Proposal Writer — Adaptive Workflow

---

# ANALYSIS PHASE

**Purpose**: Understand the RFP thoroughly and extract all requirements
**Focus**: Determine WHAT is being requested and WHAT constraints exist
**Stages in ANALYSIS PHASE**:
- RFP Intake (ALWAYS)
- Requirements Extraction (ALWAYS)
- Compliance Matrix (ALWAYS)

---

## RFP Intake (ALWAYS EXECUTE)

1. **MANDATORY**: Log initial user request and RFP document in audit.md with complete raw input
2. Load all steps from `analysis/rfp-intake.md`
3. Execute RFP intake:
   - Read and parse the full RFP document
   - Identify document structure (sections, appendices, attachments)
   - Determine RFP type (IT services, consulting, construction, etc.)
   - Extract key dates (submission deadline, Q&A period, evaluation timeline)
   - Identify issuing organization and contact information
4. **MANDATORY**: Log findings in audit.md
5. Present completion message to user (see rfp-intake.md for message formats)
6. Automatically proceed to Requirements Extraction

## Requirements Extraction (ALWAYS EXECUTE)

**Always executes** but depth varies based on RFP complexity:
- **Minimal**: Simple RFP with clear, numbered requirements
- **Standard**: Moderate complexity with multiple evaluation areas
- **Comprehensive**: Complex RFP with cross-referenced requirements, multiple volumes

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `analysis/requirements-extraction.md`
3. Execute requirements extraction:
   - Categorize requirements (mandatory vs. desirable vs. informational)
   - Extract evaluation criteria and scoring weights
   - Identify technical requirements and specifications
   - Extract submission format requirements (page limits, font, structure)
   - Identify compliance and regulatory requirements
   - Note ambiguous or conflicting requirements for clarification
4. Execute at appropriate depth (minimal/standard/comprehensive)
5. **Wait for Explicit Approval**: Follow approval format from requirements-extraction.md — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

## Compliance Matrix (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `analysis/compliance-matrix.md`
3. **MANDATORY**: Load content validation rules from `common/content-validation.md`
4. Execute compliance matrix creation:
   - Map each RFP requirement to a compliance status (Compliant / Partial / Non-Compliant / Not Applicable)
   - Identify gaps requiring mitigation strategies
   - Cross-reference requirements with evaluation criteria
   - Flag high-risk areas (mandatory requirements with compliance gaps)
5. **MANDATORY**: Validate all content before file creation per content-validation.md rules
6. **Wait for Explicit Approval**: Present compliance matrix for review — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

---

### PHASE COMPLETE: ANALYSIS

**All stages in the Analysis phase are complete.**

### WHAT'S NEXT

The next phase is **PLANNING**: Design the proposal structure and develop win strategy.

**A) Proceed to PLANNING** — Continue the workflow
**B) Review ANALYSIS outputs** — Review before moving forward

---

# PLANNING PHASE

**Purpose**: Design the proposal structure and develop a winning approach
**Focus**: Determine HOW to respond and WHAT differentiates our proposal
**Stages in PLANNING PHASE**:
- Proposal Structure Design (ALWAYS)
- Win Strategy (CONDITIONAL — Competitive bid only)
- Content Planning (ALWAYS)

---

## Proposal Structure Design (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `planning/proposal-structure-design.md`
3. Execute proposal structure design:
   - Design table of contents based on RFP requirements
   - Map proposal sections to RFP evaluation criteria
   - Determine volume structure (if multi-volume response required)
   - Allocate page budgets per section (if page limits specified)
   - Define cross-reference strategy between sections
4. **Wait for Explicit Approval**: Present proposed structure for review — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

## Win Strategy (CONDITIONAL EXECUTE)

**Execute IF**:
- Competitive bid (multiple vendors expected)
- Evaluation criteria include qualitative scoring
- User requests differentiation strategy
- RFP includes oral presentation or demonstration phase

**Skip IF**:
- Sole source procurement
- User explicitly declines strategy development
- RFP is purely price-based evaluation

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `planning/win-strategy.md`
3. Execute win strategy development:
   - Identify key win themes (2-4 themes)
   - Map themes to evaluation criteria
   - Define discriminators vs. competitors
   - Develop ghost strategy (address competitor weaknesses)
   - Create theme integration plan for each proposal section
4. **Wait for Explicit Approval**: Present win strategy for review — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

## Content Planning (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `planning/content-planning.md`
3. Load all prior context:
   - Requirements extraction results
   - Compliance matrix
   - Proposal structure design
   - Win strategy (if executed)
4. Execute content planning:
   - Create content outline for each proposal section
   - Assign key messages per section
   - Identify evidence and proof points needed
   - Define graphics and visual elements
   - Create section-level compliance mapping
5. **Wait for Explicit Approval**: Present content plan for review — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

---

### PHASE COMPLETE: PLANNING

**All stages in the Planning phase are complete.**

### WHAT'S NEXT

The next phase is **WRITING**: Draft all proposal sections.

**A) Proceed to WRITING** — Continue the workflow
**B) Review PLANNING outputs** — Review before moving forward

---

# WRITING PHASE

**Purpose**: Draft all proposal sections with compelling, compliant content
**Focus**: CREATE the actual proposal content
**Stages in WRITING PHASE**:
- Executive Summary (ALWAYS)
- Technical Response (ALWAYS)
- Management Response (CONDITIONAL — When required by RFP)
- Pricing Framework (CONDITIONAL — When pricing section required)

---

## Executive Summary (ALWAYS EXECUTE)

**Executive Summary has two parts within one stage**:
1. **Part 1 — Planning**: Create executive summary outline with key messages, collect feedback
2. **Part 2 — Generation**: Write the complete executive summary

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `writing/executive-summary.md`
3. **PART 1 — Planning**: Create executive summary outline, get user approval
4. **PART 2 — Generation**: Write the executive summary incorporating:
   - Opening hook aligned with customer's mission
   - Problem/need summary (from RFP analysis)
   - Solution overview (high-level approach)
   - Win themes integration (if win strategy executed)
   - Key differentiators and value proposition
   - Call to action / closing statement
5. **MANDATORY**: Present standardized 2-option completion message as defined in executive-summary.md — DO NOT use emergent 3-option behavior
6. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Technical Response (ALWAYS EXECUTE)

**Technical Response has two parts within one stage**:
1. **Part 1 — Planning**: Create technical section plan with approach outline
2. **Part 2 — Generation**: Write all technical response sections

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `writing/technical-response.md`
3. **PART 1 — Planning**: Create technical response plan, get user approval
4. **PART 2 — Generation**: Write technical response sections:
   - Technical approach and methodology
   - Solution architecture / system design
   - Implementation plan and timeline
   - Risk mitigation strategies
   - Quality assurance approach
   - Technology stack and tools
5. **MANDATORY**: Present standardized 2-option completion message as defined in technical-response.md
6. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
7. **MANDATORY**: Log user's response in audit.md with complete raw input

## Management Response (CONDITIONAL EXECUTE)

**Execute IF**:
- RFP requests management or organizational approach
- Evaluation criteria include management/staffing scoring
- Team structure or key personnel section required
- Project management methodology evaluation included

**Skip IF**:
- RFP does not request management information
- Pure technical/price evaluation only
- User explicitly skips this section

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `writing/management-response.md`
3. Write management response sections:
   - Organizational overview and qualifications
   - Project team structure and key personnel
   - Project management methodology
   - Communication and reporting plan
   - Staffing plan and resource allocation
   - Past performance and references
4. **MANDATORY**: Present standardized 2-option completion message as defined in management-response.md
5. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

## Pricing Framework (CONDITIONAL EXECUTE)

**Execute IF**:
- RFP requests cost/pricing proposal
- Evaluation criteria include price scoring
- Budget or cost structure section required

**Skip IF**:
- Pricing handled separately outside this workflow
- RFP does not request pricing information
- User explicitly skips pricing section

**Execution**:
1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `writing/pricing-framework.md`
3. Create pricing framework:
   - Pricing methodology and assumptions
   - Cost breakdown structure
   - Rate tables and labor categories
   - Optional pricing / value-added items
   - Payment terms and conditions
   - Cost narrative explaining pricing rationale
4. **MANDATORY**: Present standardized 2-option completion message as defined in pricing-framework.md
5. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Next Stage" — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

---

### PHASE COMPLETE: WRITING

**All stages in the Writing phase are complete.**

### WHAT'S NEXT

The next phase is **REVIEW**: Verify compliance and quality of the complete proposal.

**A) Proceed to REVIEW** — Continue the workflow
**B) Review WRITING outputs** — Review before moving forward

---

# REVIEW PHASE

**Purpose**: Ensure the proposal is compliant, high-quality, and ready for submission
**Focus**: VERIFY and FINALIZE the complete proposal
**Stages in REVIEW PHASE**:
- Compliance Check (ALWAYS)
- Quality Review (ALWAYS)
- Final Assembly (ALWAYS)

---

## Compliance Check (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `review/compliance-check.md`
3. Execute compliance verification:
   - Cross-reference every RFP requirement against proposal content
   - Update compliance matrix with actual response locations
   - Verify submission format requirements are met (page limits, fonts, structure)
   - Check all mandatory sections are present and complete
   - Identify any remaining compliance gaps
4. **Wait for Explicit Approval**: Present compliance check results — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

## Quality Review (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `review/quality-review.md`
3. Execute quality review:
   - Verify consistent tone and messaging across sections
   - Check win theme integration throughout proposal
   - Review technical accuracy and feasibility
   - Verify graphics and tables are referenced correctly
   - Check for spelling, grammar, and formatting consistency
   - Validate cross-references between sections
4. **MANDATORY**: Present standardized 2-option completion message as defined in quality-review.md
5. **Wait for Explicit Approval**: User must choose between "Request Changes" or "Continue to Final Assembly" — DO NOT PROCEED until user confirms
6. **MANDATORY**: Log user's response in audit.md with complete raw input

## Final Assembly (ALWAYS EXECUTE)

1. **MANDATORY**: Log any user input during this stage in audit.md
2. Load all steps from `review/final-assembly.md`
3. Execute final assembly:
   - Compile all sections into final proposal document structure
   - Generate table of contents with accurate page references
   - Add required headers, footers, and page numbers
   - Include all appendices and attachments
   - Create submission checklist
   - Generate final proposal summary
4. **Wait for Explicit Approval**: Present final proposal for review — DO NOT PROCEED until user confirms
5. **MANDATORY**: Log user's response in audit.md with complete raw input

---

### PROPOSAL COMPLETE

**The proposal for [RFP Title] has been generated and reviewed.**

**Final Deliverables:**
- Directory: `proposal-docs/`
- Compliance matrix: Complete with response mappings
- Proposal sections: All required volumes/sections
- Quality review: Passed

**Contents:**
- Executive Summary
- Technical Response
- Management Response (if applicable)
- Pricing Framework (if applicable)
- Compliance Matrix
- Submission Checklist

### NEXT STEPS

1. Review the complete proposal in `proposal-docs/`
2. Conduct internal review with subject matter experts
3. Prepare final submission package per RFP instructions
4. Submit before the deadline

---

## Key Principles

- **Compliance First**: Every RFP requirement must be addressed — compliance is non-negotiable
- **Customer-Centric**: Write from the evaluator's perspective, making scoring easy
- **Evidence-Based**: Support claims with specific examples, metrics, and proof points
- **Win Theme Integration**: Reinforce differentiators consistently across all sections
- **Adaptive Execution**: Only execute stages that add value based on RFP requirements
- **Transparent Planning**: Always show execution plan before starting
- **User Control**: User can request stage inclusion/exclusion
- **Progress Tracking**: Update proposal-state.md with executed and skipped stages
- **Complete Audit Trail**: Log ALL user inputs and AI responses in audit.md with timestamps
  - **CRITICAL**: Capture user's COMPLETE RAW INPUT exactly as provided
  - **CRITICAL**: Never summarize or paraphrase user input in audit log
  - **CRITICAL**: Log every interaction, not just approvals
- **Content Validation**: Always validate content before file creation per content-validation.md rules
- **NO EMERGENT BEHAVIOR**: All stages MUST use standardized 2-option completion messages as defined in their respective rule files. DO NOT create 3-option menus or other emergent navigation patterns.

## MANDATORY: Plan-Level Checkbox Enforcement

### MANDATORY RULES FOR PLAN EXECUTION
1. **NEVER complete any work without updating plan checkboxes**
2. **IMMEDIATELY after completing ANY step described in a plan file, mark that step [x]**
3. **This must happen in the SAME interaction where the work is completed**
4. **NO EXCEPTIONS**: Every plan step completion MUST be tracked with checkbox updates

### Two-Level Checkbox Tracking System
- **Plan-Level**: Track detailed execution progress within each stage
- **Stage-Level**: Track overall workflow progress in proposal-state.md
- **Update immediately**: All progress updates in SAME interaction where work is completed

## Prompts Logging Requirements
- **MANDATORY**: Log EVERY user input (prompts, questions, responses) with timestamp in audit.md
- **MANDATORY**: Capture user's COMPLETE RAW INPUT exactly as provided (never summarize)
- **MANDATORY**: Log every approval prompt with timestamp before asking the user
- **MANDATORY**: Record every user response with timestamp after receiving it
- **CRITICAL**: ALWAYS append changes to EDIT audit.md file, NEVER use tools and commands that completely overwrite its contents
- **CRITICAL**: Using file writing tools and commands that overwrite contents of the entire audit.md causes duplication
- Use ISO 8601 format for timestamps (YYYY-MM-DDTHH:MM:SSZ)
- Include stage context for each entry

### Audit Log Format:
```markdown
## [Stage Name or Interaction Type]
**Timestamp**: [ISO timestamp]
**User Input**: "[Complete raw user input - never summarized]"
**AI Response**: "[AI's response or action taken]"
**Context**: [Stage, action, or decision made]

---
```

### Correct Tool Usage for audit.md

Correct:

1. Read the audit.md file
2. Append/Edit the file to make changes

Wrong:

1. Read the audit.md file
2. Completely overwrite the audit.md with the contents of what you read, plus the new changes you want to add to it

## Directory Structure

```text
<WORKSPACE-ROOT>/
├── [RFP document(s)]                    # Input RFP files
│
├── proposal-docs/                       # PROPOSAL OUTPUT
│   ├── analysis/                        # ANALYSIS PHASE
│   │   ├── rfp-summary.md
│   │   ├── requirements.md
│   │   └── compliance-matrix.md
│   ├── planning/                        # PLANNING PHASE
│   │   ├── proposal-structure.md
│   │   ├── win-strategy.md              # If competitive bid
│   │   └── content-plan.md
│   ├── proposal/                        # WRITING PHASE
│   │   ├── executive-summary.md
│   │   ├── technical-response.md
│   │   ├── management-response.md       # If required
│   │   └── pricing-framework.md         # If required
│   ├── review/                          # REVIEW PHASE
│   │   ├── compliance-check-report.md
│   │   ├── quality-review-report.md
│   │   └── submission-checklist.md
│   ├── proposal-state.md
│   └── audit.md
```

**CRITICAL RULE**:
- RFP source documents: Workspace root or user-specified location
- All proposal outputs: `proposal-docs/` directory only
- State and audit files: `proposal-docs/` root level
