# Automated Validation — PACKAGING Phase

## Purpose
Execute 3-layer testing on the complete plugin structure to verify it is ready for distribution. This is the plugin-specific counterpart to G4 Integration Validation — it validates the plugin packaging, not just the policy content.

## Prerequisites
- Plugin Structure Generation (P1) completed and approved
- All plugin files generated (plugin.json, agents/, skills/, commands/, rules/, rule-details/)
- `common/content-validation.md` loaded for validation rules

## Execution Classification
**Type**: ALWAYS
This stage always executes as automated validation is mandatory before delivery.

---

## Execution Steps

### Step 1: Load Plugin Structure
**Action**: Load all files generated in P1
**Input**: Plugin directory structure
**Output**: Complete file inventory

Build inventory:
- [ ] plugin.json loaded and parsed
- [ ] marketplace.json loaded and parsed
- [ ] All agent definition files listed
- [ ] All SKILL.md files listed
- [ ] All command files listed
- [ ] All rules files listed
- [ ] All rule-details files listed

### Step 2: Execute Structure Tests

Verify the plugin directory is well-formed:

| Check | Method | Pass Criteria |
|-------|--------|---------------|
| plugin.json required fields | Parse JSON, verify name/version/description/agents/skills/commands | All present |
| File reference resolution | For each path in plugin.json, verify file exists | 100% resolved |
| Directory structure | Verify expected directories exist | All present |
| Rule-details completeness | Compare rule-details/ against generated policy file list | All files copied |
| Markdown syntax | Validate all .md files for heading hierarchy, link integrity | 0 errors |

**FAIL Action**: Return to P1 to fix structural issues.

### Step 3: Execute Content Tests

Verify file content meets quality standards:

| Check | Method | Pass Criteria |
|-------|--------|---------------|
| SKILL.md line count | `wc -l` each SKILL.md | 75-150 lines each |
| core-workflow line count | `wc -l` core-workflow.md | 200-600 lines |
| Phase Rule line count | `wc -l` each phase rule file | 100-300 lines each |
| Common Rule line count | `wc -l` each common rule file | 50-200 lines each |
| Agent definition sections | Check for Role, Capabilities, Workflow Integration | All sections present |
| Command definition sections | Check for Execution steps, Prerequisites | All sections present |

**FAIL Action**: Fix line count violations (trim or expand), add missing sections.

### Step 4: Execute Smoke Tests

Verify the workflow is executable for all 4 agent types:

**For each agent type** (Process, Task, Analytical, Hybrid):

1. **Entry Point Test**: Simulate invoking the main skill — does it load core-workflow.md?
2. **Flow Traversal**: Starting from Phase 1 Stage 1, can every stage be reached by following the workflow?
3. **Common Rules Loading**: Are all 11 common rules referenced and loadable?
4. **Approval Gate Test**: Does every stage end with a 2-option completion message?
5. **CONDITIONAL Path Test**: Are CONDITIONAL stages properly gated with Execute IF / Skip IF?

| Agent Type | Expected Phases | Expected Stages | Key Variation |
|-----------|----------------|-----------------|---------------|
| Process | 4-5 | 15-18 | Full workflow with all checkpoints |
| Task | 2-3 | 6-10 | Shortened workflow, fewer conditional stages |
| Analytical | 2-4 | 8-12 | Template-heavy, validation-focused |
| Hybrid | 3-6 | Variable | Mode-switching between patterns |

**FAIL Action**: Identify blocking inconsistency, determine if issue is policy-side (→ repair loop to G1/G3) or plugin-side (→ return to P1).

### Step 5: Generate Validation Report

```markdown
## Plugin Validation Report

### Structure Tests
- Total checks: [N]
- PASS: [N]
- FAIL: [N]
  - [FAIL item]: [details]

### Content Tests
- SKILL.md files: [N] checked, [N] in range (75-150)
- core-workflow: [N] lines ([in range / out of range])
- Phase Rules: [N] checked, [N] in range (100-300)
- Common Rules: [N] checked, [N] in range (50-200)
- Agent definitions: [N] complete / [N] total
- Command definitions: [N] complete / [N] total

### Smoke Tests
- Agent Types tested: [N]/4
  - Process: [PASS/FAIL] — [details if FAIL]
  - Task: [PASS/FAIL] — [details if FAIL]
  - Analytical: [PASS/FAIL] — [details if FAIL]
  - Hybrid: [PASS/FAIL] — [details if FAIL]

### Overall: [PASS/FAIL]
```

### Step 6: Present Results

**If PASS**: Present validation report, proceed to P3 or Delivery.
**If FAIL**: Present validation report with specific failures, return to P1 for targeted fixes (max 2 P2→P1 loops).

---

## Loop Control

| Rule | Limit |
|------|-------|
| P2→P1 loop | Max 2 times (separate from workflow-wide 3-count) |
| Escalation | On 2nd FAIL, present options: fix specific issues / deliver with quality note / rescope |

---

## Output Artifacts

### Validation Report
- **File**: `steering-docs/<agent-name>/packaging/plugin-validation-report.md`
- **Content**: 3-layer test results with overall PASS/FAIL

---

## Completion Message

### REVIEW REQUIRED

**Automated Validation is complete.**

- **Structure Tests**: [PASS/FAIL] ([N] checks)
- **Content Tests**: [PASS/FAIL] ([N] files validated)
- **Smoke Tests**: [PASS/FAIL] ([N]/4 agent types tested)
- **Overall**: [PASS/FAIL]

### WHAT'S NEXT

**A) Request Changes** — I'll address specific validation failures
**B) Continue to Migration Execution** — Proceed to migrate legacy structure (if applicable)

---

## Error Handling

### Structure Test FAIL
- **Severity**: High
- **Solution**: Return to P1 to fix missing files or broken references
- **Recovery**: Regenerate specific files, re-run structure tests only

### Content Test FAIL — Line Count Out of Range
- **Severity**: Medium
- **Solution**: Trim over-length files (delegate to rule-details) or expand under-length files
- **Recovery**: Edit specific files, re-run content tests only

### Smoke Test FAIL — Flow Break
- **Severity**: Critical
- **Solution**: Cross-reference with G4 Integration Validation results
  - If policy issue → repair loop to G1 or G3 (counts toward workflow-wide 3-count)
  - If plugin structure issue → return to P1 (counts toward P2→P1 2-count)
- **Recovery**: Fix specific flow break, re-run full smoke test

### Smoke Test FAIL — Specific Agent Type Only
- **Severity**: High
- **Solution**: Check agent-type-specific templates and branching logic
- **Recovery**: Fix the specific type's handling, re-test that type only

## References
- `common/content-validation.md` — Plugin structure validation rules
- `common/implementation-knowhow.md` — Skill design patterns
- `common/output-structure-patterns.md` — File patterns and line count guidelines
- `generation/integration-validation.md` — G4 validation for cross-reference
