# Common Rules Generation — GENERATION Phase

## Purpose
Generate all common (cross-phase) rule files for the target agent's steering policy set. Each common rule file must be adapted to the target agent's domain while maintaining the structural patterns established by the SPM template.

## Prerequisites
- All DISCOVERY phase artifacts available and approved
- All DESIGN phase artifacts available and approved
- Core Workflow Generation completed and approved
- Common Rules Design document (from DESIGN phase) specifying adaptation levels

## Execution Classification
**Type**: ALWAYS
This stage always executes as common rules are fundamental to every steering policy set.

---

## Execution Steps

### Step 1: Load Generation Context
**Action**: Load all relevant prior artifacts
**Input**: Design artifacts + generated core workflow
**Output**: Complete generation context

Key context to load:
- Common Rules Design (adaptation levels and specifications)
- Domain Research (terminology, best practices, pitfalls)
- Workflow Architecture (phase/stage structure for process overview)
- Quality Mechanisms Design (for quality standards and error handling)
- Generated core workflow (for cross-reference consistency)

### Step 2: Establish Generation Order
**Action**: Determine the order for generating common rule files
**Input**: Common rules inventory + cross-dependencies
**Output**: Ordered generation list

**Recommended Generation Order** (dependencies flow forward):

1. **`terminology.md`** — All other files reference terms defined here
2. **`question-format-guide.md`** — All phase files reference question formatting
3. **`content-validation.md`** — Referenced by generation and validation stages
4. **`overconfidence-prevention.md`** — Referenced by all question-generating stages
5. **`error-handling.md`** — Referenced by all stages for error recovery
6. **`session-continuity.md`** — Defines state tracking referenced by core workflow
7. **`quality-standards.md`** — Defines quality benchmarks for validation
8. **`output-structure-patterns.md`** — Defines file patterns for all generated files
9. **`process-overview.md`** — References all phases and stages
10. **`welcome-message.md`** — References phases and provides introduction

Then generate any domain-specific common rules (11+).

### Step 3: Generate Each Common Rule File
**Action**: Generate each file following its adaptation specification
**Input**: Common Rules Design + domain context + SPM templates
**Output**: Generated common rule files

#### For Each Common Rule File:

##### 3a. Review Adaptation Specification
- What adaptation level was specified? (Light/Medium/Heavy)
- What domain-specific content is needed?
- What SPM template patterns should be followed?
- What cross-references are needed?

##### 3b. Generate Content
Apply the adaptation level:

**Light Adaptation**:
- Start with SPM template structure
- Replace examples with domain-specific ones
- Update terminology to match domain glossary
- Adjust file references to target agent's structure

**Medium Adaptation**:
- Use SPM template as structural guide
- Write significant domain-specific content sections
- Add domain-specific rules, patterns, and scenarios
- Maintain SPM structural patterns but customize deeply

**Heavy Adaptation**:
- Use SPM template as inspiration only
- Write primarily domain-specific content
- Structure inspired by but not copied from template
- Ensure all required sections are present

##### 3c. Validate Content
Before writing each file:
- [ ] Markdown syntax correct
- [ ] All cross-references valid
- [ ] Domain terminology matches terminology.md
- [ ] Structural pattern matches output-structure-patterns.md
- [ ] Content is domain-specific (not generic)
- [ ] Required sections present (Purpose, main content, error handling, references)

##### 3d. Write File
Write to: `<agent-name>/skills/<skill-name>/rule-details/common/<filename>.md`

### Step 4: File-by-File Generation Guide

#### File 1: `terminology.md`
**Adaptation Level**: Heavy
**Key Content**:
- Phase/Stage hierarchy specific to target agent
- All domain-specific terms from domain research
- Agent-type-specific terminology
- Execution classifications used
- Artifact types and abbreviations

**Must Include**:
- Correct/incorrect usage examples
- Clear hierarchy definitions
- All abbreviations used in the policy set

#### File 2: `question-format-guide.md`
**Adaptation Level**: Light-Medium
**Key Content**:
- Standard question file format (mostly template)
- Domain-specific question examples
- Domain-specific contradiction patterns
- Adapted file naming for target agent's phases

**Must Include**:
- Multiple choice format with mandatory "Other"
- [Answer]: tag format
- Contradiction detection rules
- Clarification question workflow

#### File 3: `content-validation.md`
**Adaptation Level**: Medium
**Key Content**:
- Standard validation rules (from template)
- Domain-specific content types and validation
- Cross-reference validation for target agent's file structure
- Domain-specific diagram or output validation

**Must Include**:
- Pre-creation validation checklist
- Fallback strategies
- Error prevention rules

#### File 4: `overconfidence-prevention.md`
**Adaptation Level**: Medium
**Key Content**:
- "When in doubt, ask" philosophy
- Domain-specific overconfidence scenarios
- Phase-specific guidance for target agent's phases
- Domain-specific red flags

**Must Include**:
- Question generation guidelines
- Answer analysis requirements
- Red flags and success indicators

#### File 5: `error-handling.md`
**Adaptation Level**: Heavy
**Key Content**:
- Error severity levels
- Phase-specific error scenarios for EACH phase of target agent
- Recovery procedures for target agent's workflow
- Session resumption error handling

**Must Include**:
- All phases covered with specific error scenarios
- Recovery steps for each error type
- Escalation guidelines
- Error and recovery logging formats

#### File 6: `session-continuity.md`
**Adaptation Level**: Medium
**Key Content**:
- Welcome back prompt adapted for target agent
- State file format for target agent's workflow
- Context loading priorities per phase
- Artifact loading requirements per stage

**Must Include**:
- State file format with all phases/stages
- Loading rules per phase
- Error handling for resumption issues

#### File 7: `quality-standards.md`
**Adaptation Level**: Medium-Heavy
**Key Content**:
- Quality dimensions adapted to target agent's domain
- Domain-specific quality benchmarks
- Quantitative standards (file count, line count expectations)
- Quality calibration scoring format

**Must Include**:
- All relevant quality dimensions with domain-specific criteria
- Pass/Partial/Fail evaluation criteria
- Scoring format

#### File 8: `output-structure-patterns.md`
**Adaptation Level**: Medium
**Key Content**:
- Core workflow file pattern (adapted to target agent)
- Common rule file pattern
- Phase rule file pattern (adapted to target agent's phases)
- Question file, plan file, report file patterns
- Completion message patterns

**Must Include**:
- All file type patterns with examples
- State and audit file patterns
- Completion message templates (2-option standard)

#### File 9: `process-overview.md`
**Adaptation Level**: Heavy
**Key Content**:
- Complete workflow overview for target agent
- Mermaid diagram showing all phases and stages
- Stage descriptions with ALWAYS/CONDITIONAL labels
- Key principles

**Must Include**:
- Workflow diagram (Mermaid + text fallback)
- All phases and stages described
- ALWAYS vs CONDITIONAL visual distinction

#### File 10: `welcome-message.md`
**Adaptation Level**: Heavy
**Key Content**:
- Agent introduction (what it does, how it helps)
- ASCII workflow diagram showing target agent's phases
- Phase breakdown descriptions
- Key principles
- "What happens next" section

**Must Include**:
- Clear, user-friendly introduction
- Visual workflow representation
- Getting started guidance

### Step 5: Cross-Reference Validation
**Action**: Validate all cross-references between generated common rule files
**Input**: All generated common rule files
**Output**: Cross-reference validation results

Check:
- All file references between common rules resolve
- All terminology references match terminology.md
- All patterns referenced match output-structure-patterns.md
- Process overview matches actual workflow structure

### Step 6: Present Generated Files for Review
**Action**: Present all generated common rule files
**Input**: All generated files + validation results
**Output**: Summary for user review

---

## Output Artifacts

### Generated Common Rule Files
- **Location**: `<agent-name>/skills/<skill-name>/rule-details/common/`
- **Count**: 10 standard + [N] domain-specific
- **Total Lines**: Estimated [N] lines across all files

---

## Completion Message

### REVIEW REQUIRED

**Common Rules Generation is complete.** Here's what was produced:

- **Files Generated**: [N] common rule files
- **Location**: `<agent-name>/skills/<skill-name>/rule-details/common/`
- **Total Lines**: [N]
- **Cross-Reference Validation**: [PASS/FAIL]

Generated files:
1. `terminology.md` — [N] lines
2. `question-format-guide.md` — [N] lines
3. `content-validation.md` — [N] lines
4. `overconfidence-prevention.md` — [N] lines
5. `error-handling.md` — [N] lines
6. `session-continuity.md` — [N] lines
7. `quality-standards.md` — [N] lines
8. `output-structure-patterns.md` — [N] lines
9. `process-overview.md` — [N] lines
10. `welcome-message.md` — [N] lines
[11+. Domain-specific files]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the common rules based on your feedback
**B) Continue to Phase Rules Generation** — Proceed to generate phase-specific rule files

---

## Error Handling

### Domain-Specific Content Insufficient
- **Issue**: Generated file is too generic, lacks domain specificity
- **Solution**: Review domain research, add specific terminology, examples, and scenarios
- **Do Not Proceed**: Until files contain meaningful domain-specific content

### Cross-Reference Failure
- **Issue**: Common rule files reference non-existent files or sections
- **Solution**: Fix references, may need to generate referenced content
- **Recovery**: Re-validate after fixes

### File Too Short
- **Issue**: Generated file lacks sufficient depth for the adaptation level
- **Solution**: Expand with additional domain-specific content, examples, and scenarios
- **Benchmark**: Light adaptation ~50-100 lines, Medium ~100-200, Heavy ~150-300

### Consistency Conflict
- **Issue**: Content in one common rule contradicts another
- **Solution**: Align content using terminology.md as the source of truth
- **Recovery**: Fix the less-established file to match the more-established one

## References
- `design/common-rules-design.md` — Adaptation specifications
- `generation/core-workflow-generation.md` — Core workflow for consistency
- `common/content-validation.md` — Validation rules
- `common/output-structure-patterns.md` — File patterns
