# Error Handling and Recovery Procedures

## General Error Handling Principles

### When Errors Occur
1. **Identify the error**: Clearly state what went wrong
2. **Assess impact**: Determine if the error is blocking or can be worked around
3. **Communicate**: Inform the user about the error and options
4. **Offer solutions**: Provide clear steps to resolve or work around the error
5. **Document**: Log the error and resolution in `audit.md`

### Error Severity Levels

**Critical**: Workflow cannot continue
- Missing required DISCOVERY phase artifacts when entering DESIGN
- User input that fundamentally contradicts the established agent classification
- System errors preventing file operations
- Generated policy files that are internally inconsistent

**High**: Phase cannot complete as planned
- Incomplete answers to required questions
- Contradictory user responses within a phase
- Missing dependencies from prior phases
- Domain research reveals fundamental issues with the current approach

**Medium**: Phase can continue with workarounds
- Optional artifacts missing
- Non-critical validation failures
- Partial completion possible
- Minor inconsistencies in generated content

**Low**: Minor issues that don't block progress
- Formatting inconsistencies
- Optional information missing
- Non-blocking warnings
- Stylistic concerns in generated files

---

## Phase-Specific Error Handling

### DISCOVERY Phase Errors

#### Purpose Analysis Errors

**Error**: User's description is too vague to classify agent type
- **Cause**: Insufficient initial input
- **Solution**: Create detailed purpose-analysis-questions.md with classification options
- **Do Not Proceed**: Until agent type is confirmed by user

**Error**: Agent purpose doesn't fit any standard classification
- **Cause**: Novel agent type or unusual combination
- **Solution**: Present Hybrid classification with detailed breakdown of which aspects match which type
- **Escalation**: Ask user to describe ideal workflow pattern

**Error**: User changes purpose mid-analysis
- **Cause**: Evolving understanding, scope change
- **Solution**: Log the change in audit.md, restart purpose analysis with new input
- **Recovery**: Preserve any still-relevant findings from previous analysis

#### Domain Research Errors

**Error**: Domain is unfamiliar or highly specialized
- **Cause**: Novel domain, niche industry
- **Solution**: Switch to comprehensive depth, ask user for domain references and standards
- **Workaround**: Ask user to provide domain documentation or reference materials

**Error**: Conflicting best practices found during research
- **Cause**: Domain has competing standards or approaches
- **Solution**: Present alternatives to user with pros/cons, ask for preference
- **Do Not Proceed**: Until user selects preferred approach

**Error**: Domain research reveals the target agent needs are significantly different from initial classification
- **Cause**: Surface-level vs deep understanding mismatch
- **Solution**: Return to Purpose Analysis with new information, re-classify
- **Recovery**: Preserve domain research findings for use in updated classification

#### Scope Definition Errors

**Error**: Estimated scope is too large for practical generation
- **Cause**: Complex domain, many stakeholders, ambitious requirements
- **Solution**: Present scope reduction options, suggest phased approach
- **Workaround**: Generate core rules first, add optional rules in later iterations

**Error**: Scope boundaries are unclear
- **Cause**: Overlapping responsibilities, unclear agent boundaries
- **Solution**: Create scope-definition-questions.md with boundary clarification options
- **Do Not Proceed**: Until boundaries are clearly defined

---

### DESIGN Phase Errors

#### Workflow Architecture Errors

**Error**: Cannot determine appropriate phase structure
- **Cause**: Novel agent pattern, conflicting requirements
- **Solution**: Present 2-3 architecture options with trade-offs, ask user to choose
- **Do Not Proceed**: Until architecture is approved

**Error**: Designed workflow has circular dependencies
- **Cause**: Stages that mutually depend on each other
- **Solution**: Identify circular dependencies, suggest restructuring
- **Recovery**: Break cycles by introducing intermediate stages or merging stages

**Error**: Workflow complexity exceeds practical limits
- **Cause**: Too many conditional branches, excessive stages
- **Solution**: Suggest simplification options, merge related stages
- **Workaround**: Create a simplified workflow with extension points for future complexity

#### Common Rules Design Errors

**Error**: Standard common rule doesn't apply to domain
- **Cause**: Domain has fundamentally different paradigm
- **Solution**: Design domain-specific replacement rule, document why standard rule was replaced
- **Recovery**: Create custom rule file with domain-appropriate content

**Error**: Missing domain-specific common rules identified late
- **Cause**: Incomplete domain research, new requirements surface
- **Solution**: Add to common rules design, update rule inventory
- **Recovery**: Return to domain research if gap is significant

#### Phase Rules Design Errors

**Error**: Rule file design is too vague for generation
- **Cause**: Insufficient detail in design specifications
- **Solution**: Expand design with specific content outlines, ask user for detail preferences
- **Do Not Proceed**: Until designs are detailed enough for content generation

**Error**: Inter-stage dependencies are unclear
- **Cause**: Complex workflow with many conditional paths
- **Solution**: Create dependency matrix, validate with user
- **Recovery**: Simplify dependencies or add explicit documentation

---

### GENERATION Phase Errors

#### Core Workflow Generation Errors

**Error**: Generated core workflow references non-existent files
- **Cause**: File list mismatch between design and generation
- **Solution**: Update references, add missing files to generation plan
- **Validation**: Verify all references before marking generation complete

**Error**: Generated workflow doesn't match approved design
- **Cause**: Drift during generation, misinterpretation of design
- **Solution**: Compare generated file against design artifact, fix discrepancies
- **Recovery**: Regenerate sections that diverged from design

#### Rule File Generation Errors

**Error**: Generated rule file is too generic
- **Cause**: Insufficient domain-specific content, template-based generation
- **Solution**: Enhance with domain-specific terminology, examples, and scenarios
- **Do Not Proceed**: Until content is sufficiently domain-specific

**Error**: Cannot generate content for specialized domain topic
- **Cause**: Domain knowledge gap
- **Solution**: Ask user for domain-specific input, reference materials, or examples
- **Workaround**: Generate structure with placeholders, mark for user completion

**Error**: Generated file exceeds practical length
- **Cause**: Too much detail, scope creep in single file
- **Solution**: Split into multiple focused files, update references
- **Recovery**: Restructure file list, update core workflow references

#### Domain Content Injection Failure (G3)

**Error**: Domain specificity rate < 40% detected during Phase Rules Generation
- **Cause**: Insufficient domain content injection, template-heavy generation
- **Severity**: High
- **Solution**: Reload Domain Research Summary, execute injection heuristics (Steps 3c-1 through 3c-5), add domain-specific examples and procedures
- **Do Not Proceed**: To G4 until domain specificity pre-check passes on all flagged files

#### Integration Validation Errors

**Error**: Cross-reference validation fails
- **Cause**: Missing files, incorrect paths, typos in references
- **Solution**: Fix broken references, generate missing files
- **Do Not Proceed**: Until all references resolve

**Error**: Flow validation reveals dead-end paths
- **Cause**: Incomplete stage coverage, missing conditional handling
- **Solution**: Add missing stage rules or connect dead-end paths to existing stages
- **Recovery**: Update workflow architecture if structural change needed

---

### REFINEMENT Phase Errors

#### Completeness Review Errors

**Error**: Significant gaps found in policy coverage
- **Cause**: Missing rule files, incomplete stage coverage
- **Solution**: Return to GENERATION phase to fill gaps
- **Recovery**: Prioritize critical gaps, address minor gaps in consistency review

#### Consistency Review Errors

**Error**: Terminology inconsistencies across multiple files
- **Cause**: Different terms used for same concept, evolving terminology during generation
- **Solution**: Update all files to match terminology.md definitions
- **Recovery**: Batch fix terminology issues, re-validate

**Error**: Structural inconsistencies across files
- **Cause**: Different patterns used in different phases
- **Solution**: Standardize to the pattern defined in output-structure-patterns.md
- **Recovery**: Reformat non-compliant files

#### Quality Calibration Errors

**Error**: Multiple quality dimensions fail
- **Cause**: Systemic quality issues, rushed generation
- **Solution**: Prioritize dimensions by impact, return to appropriate phase
- **Recovery**: Address root cause (often in DESIGN phase), regenerate affected files

**Error**: Quality calibration reveals fundamental design flaw
- **Cause**: Design decisions that undermine quality at scale
- **Solution**: Return to DESIGN phase, revise the affected design decisions
- **Recovery**: Regenerate affected files after design revision

---

### PACKAGING Phase Errors

#### Plugin Structure Generation Errors

**Error**: plugin.json required fields missing
- **Severity**: High
- **Solution**: Regenerate from template with all required fields (name, version, description, agents, skills, commands)
- **Recovery**: Re-validate against plugin.json schema

**Error**: SKILL.md exceeds line limit (>150 lines)
- **Severity**: Medium
- **Solution**: Trim to core functionality, delegate details to rule-details files
- **Recovery**: Review against `implementation-knowhow.md` skill design patterns

**Error**: Agent definition inconsistent with core-workflow
- **Severity**: High
- **Solution**: Regenerate agent definition using core-workflow.md stage structure as source of truth
- **Recovery**: Cross-reference all agent responsibilities with workflow stages

**Error**: Smoke test reveals flow break for specific agent type
- **Severity**: Critical
- **Solution**: Cross-reference with Integration Validation (G4) results. If policy issue → repair loop to G1/G3. If plugin structure issue → return to P1.
- **Recovery**: Fix the specific agent type's template or branching logic

#### Migration Errors

**Error**: Source directory changes detected during migration
- **Severity**: Medium
- **Solution**: Run diff, present changes to user for manual merge/overwrite decision
- **Do Not Proceed**: Until user confirms merge strategy

**Error**: Reference path update missed
- **Severity**: High
- **Solution**: Grep all files for old path patterns, update all occurrences
- **Recovery**: Re-validate all cross-references after fix

### Repair Loop Errors

**Error**: Repair loop max retries reached (3 total)
- **Severity**: Critical
- **Solution**: Escalate to user with choices: A) Continue, B) Abort (deliver with quality note), C) Rescope
- **Do Not Proceed**: Until user responds to escalation

**Error**: Same stage targeted for 2nd repair loop
- **Severity**: High
- **Solution**: Escalate — suggest the problem classification itself may be wrong. Present alternative repair targets.
- **Recovery**: User may choose different approach or rescope

### Integration Validation 3-Layer Test Errors

**Error**: Structure test FAIL — missing files or broken references
- **Severity**: Critical
- **Solution**: Return to G1 (Core Workflow Generation) to fix structural issues
- **Recovery**: Regenerate missing files, fix broken references

**Error**: Content test FAIL — domain specificity rate < 40%
- **Severity**: High
- **Solution**: Return to G3 (Phase Rules Generation) for domain content injection
- **Recovery**: Load Domain Research Summary, inject additional domain-specific content

**Error**: Content test FAIL — examples < 2 per file
- **Severity**: Medium
- **Solution**: Return to G3 to add GOOD/BAD examples from Domain Research
- **Recovery**: Derive examples from best practices and pitfalls in domain-research-summary

**Error**: Smoke test FAIL — unreachable stages
- **Severity**: Critical
- **Solution**: Return to G1 to fix workflow flow connectivity
- **Recovery**: Trace full flow path, identify and fix disconnected stages

---

## Recovery Procedures

### Partial Phase Completion

**Scenario**: Phase was interrupted mid-execution

**Recovery Steps**:
1. Load the phase plan file
2. Identify last completed step (last [x] checkbox)
3. Resume from next uncompleted step
4. Verify all prior steps are actually complete
5. Continue execution normally

### Corrupted State File

**Scenario**: `steering-state.md` is corrupted or inconsistent

**Recovery Steps**:
1. Create backup: `steering-state.md.backup`
2. Ask user which phase they're actually on
3. Regenerate state file from scratch
4. Mark completed phases based on existing artifacts
5. Resume from current phase

### Missing Artifacts

**Scenario**: Required artifacts from prior phase are missing

**Recovery Steps**:
1. Identify which artifacts are missing
2. Determine if they can be regenerated
3. If yes: Return to that phase, regenerate artifacts
4. If no: Ask user to provide information manually
5. Document the gap in `audit.md`

### User Wants to Restart Phase

**Scenario**: User is unhappy with phase results and wants to redo

**Recovery Steps**:
1. Confirm user wants to restart (artifacts will be overwritten)
2. Archive existing artifacts with `.backup` extension
3. Reset phase status in `steering-state.md`
4. Clear phase checkboxes in plan files
5. Re-execute phase from beginning

### User Wants to Skip Phase

**Scenario**: User wants to skip a planned phase or stage

**Recovery Steps**:
1. Confirm user understands implications
2. Document skip reason in `audit.md`
3. Mark phase/stage as "SKIPPED" in `steering-state.md`
4. Proceed to next phase/stage
5. Note: May cause issues in later phases if dependencies missing

---

## Escalation Guidelines

### When to Ask for User Help

**Immediately**:
- Contradictory or ambiguous user input
- Missing required information
- Domain-specific questions AI cannot answer
- Decisions requiring business judgment

**After Attempting Resolution**:
- Repeated errors in same step
- Complex domain-specific issues
- Unusual agent patterns not covered by classifications
- Quality dimension failures with no clear fix

### When to Suggest Starting Over

**Consider Fresh Start If**:
- Multiple phases have fundamental errors
- State file is severely corrupted
- User requirements have changed significantly
- Agent classification needs to be completely revised
- Design architecture has fundamental flaws discovered late

**Before Starting Over**:
1. Archive all existing work
2. Document lessons learned
3. Identify what to preserve
4. Get user confirmation
5. Create new execution plan

---

## Logging Requirements

### Error Logging Format

```markdown
## Error — [Phase/Stage Name]
**Timestamp**: [ISO timestamp]
**Error Type**: [Critical/High/Medium/Low]
**Description**: [What went wrong]
**Cause**: [Why it happened]
**Resolution**: [How it was resolved]
**Impact**: [Effect on workflow]

---
```

### Recovery Logging Format

```markdown
## Recovery — [Phase/Stage Name]
**Timestamp**: [ISO timestamp]
**Issue**: [What needed recovery]
**Recovery Steps**: [What was done]
**Outcome**: [Result of recovery]
**Artifacts Affected**: [List of files]

---
```

## Prevention Best Practices

1. **Validate Early**: Check inputs and dependencies before starting work
2. **Checkpoint Often**: Update checkboxes immediately after completing steps
3. **Communicate Clearly**: Explain what you're doing and why
4. **Ask Questions**: Don't assume — clarify ambiguities immediately (see overconfidence-prevention.md)
5. **Document Everything**: Log all decisions and changes in `audit.md`
6. **Validate Before Writing**: Always validate content before creating files (see content-validation.md)
