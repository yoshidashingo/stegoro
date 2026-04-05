# Phase Rules Generation — GENERATION Phase

## Purpose
Generate all phase-specific rule files for the target agent's steering policy set. Each file provides detailed instructions for a specific stage in the target agent's workflow.

## Prerequisites
- All DISCOVERY phase artifacts available and approved
- All DESIGN phase artifacts available and approved
- Core Workflow Generation completed and approved
- Common Rules Generation completed and approved
- Phase Rules Design document (from DESIGN phase) with file specifications

## Execution Classification
**Type**: ALWAYS
This stage always executes as phase-specific rules are the core content of the steering policy set.

---

## Execution Steps

### Step 1: Load Generation Context
**Action**: Load all relevant prior artifacts
**Input**: All prior artifacts + generated files
**Output**: Complete generation context

Key context to load:
- Phase Rules Design (detailed specifications for each file)
- Workflow Architecture (dependencies, flow, checkpoints)
- Generated core workflow (for reference consistency)
- Generated common rules (for cross-reference accuracy)
- Domain research (for domain-specific content)
- Quality mechanisms design (for quality-related content in each file)

### Step 2: Establish Generation Order
**Action**: Determine the order for generating phase rule files
**Input**: Phase/stage structure + dependencies
**Output**: Ordered generation list

**Generation Order Principles**:
1. Generate files in workflow execution order (Phase 1 stages first, then Phase 2, etc.)
2. Within each phase, generate ALWAYS stages before CONDITIONAL stages
3. If a stage references another stage's output, generate the referenced stage first
4. Group related stages for consistency review

### Step 3: Generate Each Phase Rule File
**Action**: Generate each file following its design specification
**Input**: Phase Rules Design + domain context + common rules
**Output**: Generated phase rule files

#### For Each Phase Rule File:

##### 3a. Review Design Specification
From the Phase Rules Design document, review:
- File name and location
- Execution classification (ALWAYS/CONDITIONAL)
- Purpose statement
- Execution steps outline
- Question categories
- Output artifacts
- Error scenarios
- Completion message design

##### 3b. Domain Content Injection Heuristics (5 Steps)

**CRITICAL**: This is the core mechanism for ensuring domain specificity (Dim 12 >= 40%).

```
Step 3b-1: Load Domain Research Summary
  - Extract best practices, pitfalls, standards, terminology

Step 3b-2: Map domain content to file sections
  - Execution Steps → domain-specific actions, checks, procedures
  - Examples → GOOD/BAD examples derived from best practices / pitfalls
  - Error Handling → pitfall-derived error scenarios

Step 3b-3: Measure domain specificity rate per file
  - Count lines containing terminology.md terms ÷ total content lines
  - Target: >= 40% per Phase Rule file

Step 3b-4: Flag files below 40% for additional injection
  - Mark under-threshold files for Step 3b-5

Step 3b-5: Inject additional domain content into flagged files
  - Add more specific examples from Domain Research
  - Add domain-specific procedures and checks
  - Add pitfall-derived error scenarios
  - Re-measure after injection
```

**Minimum Requirements per Phase Rule File**:
- >= 2 GOOD/BAD examples (Dim 13)
- >= 40% domain specificity rate (Dim 12)
- Error handling references domain pitfalls (Dim 15 >= 50%)

##### 3c. Generate Content Following Standard Pattern
Use the Phase Rule File Pattern from `common/output-structure-patterns.md`:

```markdown
# [Stage Name] — [Phase Name] Phase

## Purpose
[Stage purpose from design specification]

## Prerequisites
[List from design specification]

## Execution Classification
[ALWAYS or CONDITIONAL with criteria from design]

## Adaptive Depth
[If applicable, from design specification]

---

## Execution Steps

### Step 1: [Step Title]
**Action**: [From design specification]
**Input**: [Specified inputs]
**Output**: [Specified outputs]
**Validation**: [Verification criteria]

[Domain-specific detailed instructions]
[Examples relevant to the domain]
[Quality checks specific to this step]

### Step 2: [Step Title]
[Continue for all steps from design specification]

---

## Question Generation
[Question categories from design]
[Domain-specific question templates]
[Contradiction patterns specific to this stage]

---

## Output Artifacts
[Artifact specifications from design]

---

## Completion Message

### REVIEW REQUIRED
**[Stage Name] is complete.** [Summary from design]

### WHAT'S NEXT
**A) Request Changes** — I'll revise based on your feedback
**B) Continue to [Next Stage]** — Proceed to the next stage

---

## Error Handling
[Error scenarios from design]
[Recovery procedures]
[Reference to common/error-handling.md]
```

##### 3c. Add Domain-Specific Content
For each step in the file:
- Add domain-specific instructions and considerations
- Include domain terminology (verified against terminology.md)
- Add domain-specific examples where helpful
- Include domain-specific quality checks
- Reference domain standards or best practices

##### 3d. Add Quality Mechanisms
Embed the quality mechanisms from the design:
- MANDATORY audit logging steps
- Checkpoint references
- Content validation requirements
- Overconfidence prevention triggers
- Error handling for stage-specific scenarios

##### 3e. Validate Content
Before writing each file, verify:
- [ ] Follows Phase Rule File Pattern from output-structure-patterns.md
- [ ] Purpose section present and accurate
- [ ] Prerequisites section lists all required prior artifacts
- [ ] Execution classification matches workflow architecture
- [ ] All execution steps are detailed (not just titles)
- [ ] Question generation section includes domain-specific categories
- [ ] Output artifacts section describes file format
- [ ] Completion message uses 2-option standard format
- [ ] Error handling section covers stage-specific scenarios
- [ ] All cross-references to common rules are correct
- [ ] All cross-references to other stage files are correct
- [ ] Domain-specific content is present throughout
- [ ] Terminology matches terminology.md

##### 3f. Write File
Write to: `.<agent-name>/<agent-name>-rule-details/<phase-dir>/<filename>.md`

### Step 4: Phase-by-Phase Generation

#### Phase 1: [Phase Name]
Generate files in this order:
1. [Stage 1 — ALWAYS]: `<phase-1-dir>/<stage-1>.md`
2. [Stage 2 — CONDITIONAL]: `<phase-1-dir>/<stage-2>.md`
3. [Continue for all stages]

**After Phase 1 complete**: Cross-validate all Phase 1 files for internal consistency.

#### Phase 2: [Phase Name]
[Continue for all phases]

### Step 5: Cross-Phase Validation
**Action**: Validate all phase rule files work together
**Input**: All generated phase rule files
**Output**: Cross-phase validation results

Check:
- Stage completion messages correctly reference next stages
- Stage prerequisites correctly reference prior stages
- All inter-stage artifact references are valid
- Error handling references between stages are consistent
- Terminology is consistent across all files
- Depth level adaptation is consistent

### Step 6: Two-Part Stage Verification
**Action**: Verify any two-part stages are properly structured
**Input**: Two-part stage files
**Output**: Verification results

For each two-part stage, verify:
- Part 1 (Planning) creates a plan with checkboxes
- Part 1 requires user approval before Part 2
- Part 2 (Generation) references and follows the plan
- Both parts have appropriate audit logging

### Step 7: Present Generated Files for Review
**Action**: Present all generated phase rule files
**Input**: All generated files + validation results
**Output**: Summary for user review

---

## Output Artifacts

### Generated Phase Rule Files
- **Location**: `.<agent-name>/<agent-name>-rule-details/<phase-dirs>/`
- **Count**: [N] files across [N] phase directories
- **Total Lines**: Estimated [N] lines

---

## Completion Message

### REVIEW REQUIRED

**Phase Rules Generation is complete.** Here's what was produced:

- **Phase Directories Created**: [N]
- **Files Generated**: [N] phase rule files
- **Total Lines**: [N]
- **Cross-Phase Validation**: [PASS/FAIL]

Generated by phase:
- **[Phase 1]**: [N] files ([list file names])
- **[Phase 2]**: [N] files ([list file names])
- **[Phase N]**: [N] files ([list file names])

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the phase rules based on your feedback
**B) Continue to Integration Validation** — Proceed to validate the complete policy set

---

## Error Handling

### Design Specification Insufficient
- **Issue**: Design specification lacks enough detail to generate a complete rule file
- **Solution**: Refer to domain research for additional context, expand with domain knowledge
- **Fallback**: Generate best-effort content, flag areas needing user input
- **Recovery**: May need to return to Phase Rules Design for the affected file

### Generated File Too Thin
- **Issue**: Generated file lacks sufficient depth and detail
- **Solution**: Expand each step with domain-specific instructions, examples, and quality checks
- **Benchmark**: Most phase rule files should be 100-300 lines
- **Red Flag**: Any ALWAYS stage file under 80 lines likely needs expansion

### Cross-Reference Failures
- **Issue**: Phase rule files reference non-existent common rules or stage files
- **Solution**: Fix references, generate any missing files
- **Recovery**: Re-validate after fixes

### Inconsistent Patterns Across Phases
- **Issue**: Different phases use different structural patterns for similar content
- **Solution**: Standardize using output-structure-patterns.md as the reference
- **Recovery**: Reformat non-compliant files to match the standard pattern

### Conditional Logic Gaps
- **Issue**: CONDITIONAL stage file doesn't clearly define when to execute/skip
- **Solution**: Add explicit Execute IF and Skip IF criteria with specific, testable conditions
- **Do Not Proceed**: Until all conditional logic is unambiguous

## References
- `design/phase-rules-design.md` — File specifications
- `design/workflow-architecture.md` — Stage structure and dependencies
- `generation/core-workflow-generation.md` — Core workflow for consistency
- `generation/common-rules-generation.md` — Common rules for cross-references
- `common/output-structure-patterns.md` — Phase rule file pattern
- `common/content-validation.md` — Validation rules
