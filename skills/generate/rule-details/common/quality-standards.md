# Quality Standards — AI-DLC Reference Benchmark

## Purpose

This file defines the quality standards that ALL generated steering policy sets must meet. These standards are derived from the AI-DLC reference implementation, which represents the gold standard for AI agent steering policies.

The AI-DLC reference can be found in `.aidlc/` and consists of 26+ files totaling ~4,500 lines of comprehensive steering policy content.

---

## The 15 Quality Dimensions

Every generated steering policy set MUST achieve at least a "Pass" rating on all 15 quality dimensions (11 AI-DLC standard + 4 SPM domain-specific). These dimensions are evaluated during the REFINEMENT phase's Quality Calibration stage.

### Dimension 1: Adaptive Workflow

**Standard**: All stages MUST be explicitly classified as ALWAYS or CONDITIONAL.

**Requirements**:
- Every stage in the workflow has an explicit execution classification
- CONDITIONAL stages have clear "Execute IF" and "Skip IF" criteria
- ALWAYS stages have justification for why they cannot be skipped
- Adaptive depth is supported where applicable (Minimal/Standard/Comprehensive)

**AI-DLC Reference**:
- AI-DLC classifies 16 stages across 3 phases
- ALWAYS stages: Workspace Detection, Requirements Analysis, Workflow Planning, Code Generation, Build and Test
- CONDITIONAL stages: Reverse Engineering, User Stories, Application Design, Units Generation, Functional Design, NFR Requirements, NFR Design, Infrastructure Design

**Evaluation Criteria**:
- PASS: All stages classified, conditions clearly documented
- PARTIAL: Most stages classified, some conditions vague
- FAIL: Stages not classified or conditions missing

### Dimension 2: Mandatory Checkpoints

**Standard**: Critical decisions MUST require explicit user approval before proceeding.

**Requirements**:
- Every phase transition requires user approval
- Design decisions with significant impact require approval
- Generated content requires user review before proceeding
- Users can request changes at every checkpoint
- Standardized 2-option completion messages (Request Changes / Continue)

**AI-DLC Reference**:
- AI-DLC has checkpoints at every stage transition
- Uses standardized "Wait for Explicit Approval" pattern
- 2-option completion messages: "Request Changes" or "Continue to Next Stage"
- NEVER proceeds without user confirmation

**Evaluation Criteria**:
- PASS: All critical decisions gated, standardized messages used
- PARTIAL: Most decisions gated, some messages non-standard
- FAIL: Critical decisions proceed without approval

### Dimension 3: Question File Format

**Standard**: ALL questions MUST use structured multiple-choice files with mandatory "Other" option.

**Requirements**:
- Questions NEVER asked in chat — always in dedicated files
- Multiple choice format: A, B, C, D... + "Other" as last option
- [Answer]: tag for every question
- Minimum 2 meaningful options + Other
- Contradiction and ambiguity detection after responses
- Clarification question files when issues detected

**AI-DLC Reference**:
- AI-DLC uses `{phase-name}-questions.md` file naming
- All questions have multiple choice + mandatory "Other"
- Complete answer validation workflow
- Clarification files: `{phase-name}-clarification-questions.md`

**Evaluation Criteria**:
- PASS: All questions use file format, Other always present, validation included
- PARTIAL: Most questions use files, some missing Other or validation
- FAIL: Questions asked in chat or missing structured format

### Dimension 4: Content Validation

**Standard**: ALL generated content MUST be validated before file creation.

**Requirements**:
- Mermaid diagram syntax validation
- ASCII diagram standards compliance
- Special character escaping
- Text alternatives for visual content
- Cross-reference verification
- Markdown syntax correctness

**AI-DLC Reference**:
- Separate `content-validation.md` with detailed rules
- Separate `ascii-diagram-standards.md` for diagrams
- Validation checklist before every file write
- Fallback content patterns defined

**Evaluation Criteria**:
- PASS: Validation rules defined, fallbacks provided, checklist included
- PARTIAL: Some validation defined, incomplete coverage
- FAIL: No validation rules or checklist

### Dimension 5: Audit Trail

**Standard**: Complete logging with ISO 8601 timestamps and raw user inputs.

**Requirements**:
- Every user input logged with full raw text (never summarized)
- Every AI response logged with context
- ISO 8601 timestamp format (YYYY-MM-DDTHH:MM:SSZ)
- Audit file uses append-only pattern (never overwrite)
- Stage context included with each entry

**AI-DLC Reference**:
- `audit.md` with structured format
- CRITICAL rules about never summarizing user input
- Correct tool usage guidance (append, not overwrite)
- Error and recovery logging formats defined

**Evaluation Criteria**:
- PASS: Audit format defined, raw input captured, timestamps required
- PARTIAL: Audit exists but missing some requirements
- FAIL: No audit trail defined or critical rules missing

### Dimension 6: Error Handling

**Standard**: Phase-specific recovery procedures for all error severity levels.

**Requirements**:
- Error severity classification (Critical/High/Medium/Low)
- Phase-specific error scenarios and solutions
- Recovery procedures for interrupted workflows
- Escalation guidelines for when to ask user
- Session resumption error handling

**AI-DLC Reference**:
- Comprehensive `error-handling.md` (~420 lines)
- Covers every phase with specific error scenarios
- Recovery procedures for partial completion, corruption, missing artifacts
- Clear escalation guidelines

**Evaluation Criteria**:
- PASS: All phases covered, severity levels defined, recovery procedures included
- PARTIAL: Some phases covered, incomplete recovery procedures
- FAIL: Error handling minimal or missing

### Dimension 7: Overconfidence Prevention

**Standard**: "When in doubt, ask" philosophy enforced across all phases.

**Requirements**:
- Default to asking rather than assuming
- Comprehensive question categories for each stage
- Answer analysis for vague/ambiguous/contradictory responses
- Mandatory follow-up questions for unclear responses
- Red flags identified for overconfident behavior

**AI-DLC Reference**:
- Dedicated `overconfidence-prevention.md`
- Documents root cause analysis of overconfidence issues
- Stage-specific guidance for question generation
- Red flags and success indicators defined

**Evaluation Criteria**:
- PASS: Philosophy documented, stage-specific guidance, red flags defined
- PARTIAL: General guidance without stage-specific detail
- FAIL: No overconfidence prevention measures

### Dimension 8: Depth Levels

**Standard**: Adaptive detail based on complexity — all artifacts created, detail level adapts.

**Requirements**:
- Three depth levels defined: Minimal / Standard / Comprehensive
- Criteria for selecting depth level
- Core principle: artifacts always created, detail adapts
- Examples showing different depth levels

**AI-DLC Reference**:
- Depth levels used in Requirements Analysis, User Stories, and other stages
- Factors: request clarity, complexity, scope, risk, context
- Example patterns for simple vs complex scenarios

**Evaluation Criteria**:
- PASS: Three levels defined with criteria, examples provided
- PARTIAL: Levels mentioned but criteria unclear
- FAIL: No depth level adaptation

### Dimension 9: Session Continuity

**Standard**: State tracking via state file, session resumption with context loading.

**Requirements**:
- State file tracking current position and progress
- Welcome back prompt with status summary
- Mandatory artifact loading before resumption
- Smart context loading by phase priority
- Context summary for user awareness

**AI-DLC Reference**:
- `session-continuity.md` with templates
- `aidlc-state.md` format defined
- Mandatory artifact loading per stage
- Error handling for missing/corrupted artifacts during resumption

**Evaluation Criteria**:
- PASS: State file format defined, loading rules specified, resumption template provided
- PARTIAL: Basic state tracking without loading priorities
- FAIL: No session continuity mechanism

### Dimension 10: Terminology Standardization

**Standard**: Consistent terminology enforced via glossary across all files.

**Requirements**:
- Comprehensive terminology glossary
- Domain-specific terms defined
- Usage examples (correct and incorrect)
- Clear hierarchy (Phase > Stage > Step)
- Agent type classifications defined

**AI-DLC Reference**:
- `terminology.md` with ~170 lines (4-column table format)
- Core terminology, phase descriptions, stage definitions
- Architecture terms, artifact types, abbreviations
- Usage guidelines with examples

**Evaluation Criteria**:
- PASS: Glossary comprehensive, domain terms included, usage examples provided
- PARTIAL: Basic glossary without examples or domain terms
- FAIL: No terminology glossary

### Dimension 11: Standardized Completion Messages

**Standard**: Every stage ends with REVIEW REQUIRED section followed by WHAT'S NEXT section.

**Requirements**:
- REVIEW REQUIRED section summarizing what was produced
- WHAT'S NEXT section with exactly 2 options:
  1. Request Changes — return to current stage
  2. Continue to Next Stage — proceed forward
- No emergent 3-option or variable-option patterns
- Consistent format across all stages

**AI-DLC Reference**:
- Standardized 2-option completion messages in all construction stages
- Explicit rule: "DO NOT use emergent 3-option behavior"
- Core workflow reinforces this pattern repeatedly
- Same format for all stages

**Evaluation Criteria**:
- PASS: All stages use 2-option format, no emergent patterns
- PARTIAL: Most stages compliant, some variations
- FAIL: No standardized format or emergent patterns present

---

## Domain-Specific Quality Dimensions (Dim 12-15)

### Dimension 12: Domain Specificity Rate

**Standard**: Phase Rule files must contain >= 40% domain-specific lines.

**Measurement Method**:
1. For each Phase Rule file, scan line by line
2. A line is "domain-specific" if it: (a) contains terms from terminology.md, (b) contains domain-specific proper nouns or procedures from Domain Research, (c) is NOT a generic template boilerplate line
3. Calculate: domain-specific lines ÷ (total lines - blank lines - heading lines)

**Threshold**: >= 40%
**Threshold Rationale**: Analysis of 10 existing high-quality skills showed average domain specificity of ~45% in strong skills and ~20% in weak skills. 40% is the minimum line for meaningful domain adaptation.

**Evaluation Criteria**:
- PASS: All Phase Rule files >= 40%
- PARTIAL: Average >= 40% but some files below
- FAIL: Average < 40% or majority of files below

### Dimension 13: Example Coverage

**Standard**: Each Phase Rule file must contain >= 2 concrete examples (GOOD/BAD patterns).

**Measurement Method**: Count code blocks or structured example sections containing GOOD/BAD patterns, concrete scenarios, or domain-specific illustrations per Phase Rule file.

**Threshold**: >= 2 examples per file
**Threshold Rationale**: High-quality skills (e.g., frontend-slides, market-research) average 3-5 examples per section. 2 per file is the minimum for demonstrating expected behavior.

**Evaluation Criteria**:
- PASS: All Phase Rule files have >= 2 examples
- PARTIAL: Average >= 2 but some files have 0-1
- FAIL: Majority of files have < 2 examples

### Dimension 14: Artifact Template Completeness

**Standard**: 100% of output-producing files must include artifact templates.

**Measurement Method**: Identify files that should produce output (generation/*.md, refinement/*.md). Check each for a template block showing the expected output format.

**Threshold**: = 100%
**Threshold Rationale**: Current coverage is 25% (1/4 generation files). Without templates, generation quality varies unpredictably.

**Evaluation Criteria**:
- PASS: All output-producing files have templates
- PARTIAL: >= 75% have templates
- FAIL: < 75% have templates

### Dimension 15: Pitfall Reference Rate

**Standard**: >= 50% of error handling items must reference domain-research pitfalls.

**Measurement Method**: In error handling sections, count items that explicitly cite domain-research-summary.md pitfalls (by name or file reference) ÷ total error handling items.

**Threshold**: >= 50%
**Threshold Rationale**: If domain research identifies pitfalls but error handling doesn't reference them, the research value is wasted. 50% ensures at least half of error scenarios are domain-informed.

**Evaluation Criteria**:
- PASS: >= 50% of error items reference domain pitfalls
- PARTIAL: 25-49% reference domain pitfalls
- FAIL: < 25% reference domain pitfalls

---

## Quantitative Quality Benchmarks

### File Count and Coverage
- Generated policy set should have **sufficient files** to cover all workflow paths
- Minimum for simple Task Agent: ~15 files
- Typical for Process Agent: ~25-30 files
- Common rules: ~10 files (adapted from template)
- Phase rules: ~2-5 files per phase

### Line Count and Depth
- Generated policy set should have **sufficient depth** for production use
- Minimum total: ~2,000 lines for simple agents
- Typical total: ~3,500-5,000 lines for Process Agents
- Core workflow: ~200-500 lines
- Common rule files: ~50-200 lines each
- Phase rule files: ~100-300 lines each

### Cross-Reference Integrity
- **100%** of file references in core-workflow.md must resolve
- **100%** of cross-references between rule files must resolve
- **0** orphaned files (files not referenced by any other file)
- **0** dead-end workflow paths

---

## Quality Calibration Scoring

### Scoring Format

```markdown
# Quality Calibration Scorecard

| # | Dimension | Score | Evidence | Action |
|---|-----------|-------|----------|--------|
| 1 | Adaptive Workflow | PASS/PARTIAL/FAIL | [file:line or count] | [none/repair→target] |
| 2 | Mandatory Checkpoints | PASS/PARTIAL/FAIL | [file:line or count] | |
| 3 | Question File Format | PASS/PARTIAL/FAIL | [file:line or count] | |
| 4 | Content Validation | PASS/PARTIAL/FAIL | [file:line or count] | |
| 5 | Audit Trail | PASS/PARTIAL/FAIL | [file:line or count] | |
| 6 | Error Handling | PASS/PARTIAL/FAIL | [file:line or count] | |
| 7 | Overconfidence Prevention | PASS/PARTIAL/FAIL | [file:line or count] | |
| 8 | Depth Levels | PASS/PARTIAL/FAIL | [file:line or count] | |
| 9 | Session Continuity | PASS/PARTIAL/FAIL | [file:line or count] | |
| 10 | Terminology Standardization | PASS/PARTIAL/FAIL | [file:line or count] | |
| 11 | Standardized Completion Messages | PASS/PARTIAL/FAIL | [file:line or count] | |
| 12 | Domain Specificity Rate | PASS/PARTIAL/FAIL | [N% per file] | |
| 13 | Example Coverage | PASS/PARTIAL/FAIL | [N examples/file] | |
| 14 | Artifact Template Completeness | PASS/PARTIAL/FAIL | [N/M = X%] | |
| 15 | Pitfall Reference Rate | PASS/PARTIAL/FAIL | [N/M = X%] | |

**Summary**: [X]/15 PASS — [APPROVED/CONDITIONAL/NEEDS REMEDIATION]
```

### Approval Criteria
- **APPROVED**: All 15 dimensions PASS → proceed to PACKAGING
- **CONDITIONAL APPROVAL**: 13+ dimensions PASS, FAIL only in Dim 12-15 → user decides
- **NEEDS REMEDIATION**: <=12 dimensions PASS, or any Dim 1-11 FAIL → repair loop via repair judgment tree
