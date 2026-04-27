# Proposal Writer Terminology Glossary

## Purpose

This file standardizes terminology used throughout the proposal creation workflow. Consistent terminology prevents confusion, ensures professional quality, and maintains traceability between RFP analysis and proposal content.

---

## Core Terminology

### Phase vs Stage

**Phase**: One of the four high-level workflow phases
- **ANALYSIS PHASE** — RFP Understanding & Extraction (WHAT is requested)
- **PLANNING PHASE** — Structure & Win Strategy (HOW to respond)
- **WRITING PHASE** — Proposal Content Creation (CREATE the response)
- **REVIEW PHASE** — Quality & Compliance Assurance (VERIFY and FINALIZE)

**Stage**: An individual workflow activity within a phase
- Examples: RFP Intake stage, Requirements Extraction stage, Executive Summary stage
- Each stage has specific prerequisites, steps, and outputs
- Stages can be ALWAYS-EXECUTE or CONDITIONAL

**Usage Examples**:
- "The WRITING phase contains 4 stages"
- "The Executive Summary stage is always executed"
- "We're in the PLANNING phase, executing the Win Strategy stage"

**Incorrect Usage**:
- "The Executive Summary phase" (should be "stage")
- "The WRITING stage" (should be "phase")

---

## Four-Phase Workflow

### ANALYSIS PHASE
**Purpose**: RFP understanding and requirement extraction
**Focus**: Determine WHAT is being requested
**Location**: `analysis/` directory

**Stages**:
- RFP Intake (ALWAYS)
- Requirements Extraction (ALWAYS — Adaptive depth)
- Compliance Matrix (ALWAYS)

**Outputs**: RFP summary, categorized requirements, compliance matrix

### PLANNING PHASE
**Purpose**: Proposal strategy and structure design
**Focus**: Determine HOW to respond
**Location**: `planning/` directory

**Stages**:
- Proposal Structure Design (ALWAYS)
- Win Strategy (CONDITIONAL — Competitive bids only)
- Content Planning (ALWAYS)

**Outputs**: Proposal structure, win strategy, content plan

### WRITING PHASE
**Purpose**: Proposal content creation
**Focus**: CREATE the actual content
**Location**: `writing/` directory

**Stages**:
- Executive Summary (ALWAYS)
- Technical Response (ALWAYS)
- Management Response (CONDITIONAL)
- Pricing Framework (CONDITIONAL)

**Outputs**: Complete draft proposal sections

### REVIEW PHASE
**Purpose**: Quality assurance and compliance verification
**Focus**: VERIFY and FINALIZE the proposal
**Location**: `review/` directory

**Stages**:
- Compliance Check (ALWAYS)
- Quality Review (ALWAYS)
- Final Assembly (ALWAYS)

**Outputs**: Compliance check report, quality review report, final proposal

---

## RFP and Procurement Terminology

### RFP (Request for Proposal)
A formal solicitation document from a buyer requesting proposals from potential vendors. May also appear as RFQ (Request for Quotation), RFI (Request for Information), or IFB (Invitation for Bid).

### Evaluation Criteria
The standards by which proposals are scored. May include technical merit, management approach, past performance, and price.

### Compliance Matrix
A tracking document mapping each RFP requirement to the proposal section where it is addressed, with a compliance status.

### Win Theme
A recurring message or differentiator woven throughout the proposal to emphasize a competitive advantage.

### Ghost Strategy
A technique for indirectly addressing competitor weaknesses by highlighting your strengths in areas where competitors are known to be weak.

### Discriminator
A specific, verifiable difference between your proposal and competitors' likely offerings. Stronger than a generic strength.

### Proof Point
A specific example, metric, case study, or reference that substantiates a claim made in the proposal.

---

## Proposal Content Terminology

### Executive Summary
A high-level overview of the entire proposal, typically 1-3 pages, aimed at senior decision-makers. Should standalone as a compelling summary.

### Technical Response / Technical Approach
The section describing HOW the work will be performed, including methodology, tools, architecture, and implementation approach.

### Management Response / Management Approach
The section describing WHO will perform the work and HOW it will be managed, including team structure, key personnel, and project management methodology.

### Pricing Framework / Cost Proposal
The section describing the cost structure, including rates, labor categories, pricing methodology, and total proposed price.

### Past Performance
Documented evidence of successfully completing similar work, typically including contract references, scope descriptions, and performance metrics.

---

## Compliance Terminology

### Mandatory Requirement
A requirement the proposal MUST address. Failure to comply typically results in disqualification. Often indicated by "shall" or "must" in the RFP.

### Desirable Requirement
A requirement that is preferred but not mandatory. Addressing it adds scoring value. Often indicated by "should" in the RFP.

### Informational Requirement
A requirement for the proposer to provide information without a specific compliance standard. Often indicated by "may" or "please describe."

### Compliant
The proposal fully addresses the requirement as stated.

### Partially Compliant
The proposal addresses the requirement but with limitations, exceptions, or alternative approaches that may not fully satisfy the evaluator.

### Non-Compliant
The proposal does not address the requirement or takes explicit exception to it.

### Exception / Deviation
A formal statement that the proposer cannot or chooses not to comply with a specific requirement, typically with an explanation and alternative approach.

---

## Depth Levels

- **Minimal**: Quick, focused execution for simple, well-structured RFPs
- **Standard**: Normal depth for typical RFPs with clear evaluation criteria
- **Comprehensive**: Full depth for complex, high-value, multi-volume RFPs

---

## Artifact Types

### Plans
Documents with checkboxes and outlines that guide content creation.
- Located in `proposal-docs/planning/`
- Examples: `proposal-structure.md`, `content-plan.md`

### Artifacts
Generated outputs from executing workflow stages.
- Located in various `proposal-docs/` subdirectories
- Examples: `requirements.md`, `compliance-matrix.md`, `executive-summary.md`

### State Files
Files tracking workflow progress and status.
- `proposal-state.md`: Overall workflow state
- `audit.md`: Complete audit trail of all interactions

---

## Common Abbreviations

- **RFP**: Request for Proposal
- **RFQ**: Request for Quotation
- **RFI**: Request for Information
- **IFB**: Invitation for Bid
- **SOW**: Statement of Work
- **SOO**: Statement of Objectives
- **PWS**: Performance Work Statement
- **CLIN**: Contract Line Item Number
- **LOE**: Level of Effort
- **T&M**: Time and Materials
- **FFP**: Firm Fixed Price
- **CPFF**: Cost Plus Fixed Fee
- **KPI**: Key Performance Indicator
- **SLA**: Service Level Agreement
- **PMO**: Project Management Office
- **SME**: Subject Matter Expert
- **QA**: Quality Assurance

---

## Terminology Guidelines

### Consistency Rules
1. Use the same term for the same concept throughout all proposal documents
2. Match RFP terminology — use the buyer's words, not synonyms
3. Define acronyms on first use in each major section
4. Maintain a project-specific glossary for domain terms

### When RFP and Standard Terminology Conflict
- **Always prefer the RFP's terminology** in proposal content
- Use standard terminology in internal workflow documents (audit.md, state files)
- Map RFP terms to standard terms in the terminology section of the proposal
