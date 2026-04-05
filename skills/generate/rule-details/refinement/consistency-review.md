# Consistency Review — REFINEMENT Phase

## Purpose
Verify that the generated steering policy set maintains internal consistency across all files — terminology, structural patterns, writing style, formatting, and cross-references are uniform throughout. Inconsistencies confuse the target agent and degrade policy quality.

## Prerequisites
- Completeness Review completed and passed (or gaps remediated)
- All generated policy files written to disk
- terminology.md and output-structure-patterns.md available as references

## Execution Classification
**Type**: ALWAYS
This stage always executes as consistency is essential for the target agent to operate correctly.

---

## Execution Steps

### Step 1: Load All Generated Files
**Action**: Load every generated file for cross-file analysis
**Input**: All generated policy files
**Output**: Complete file set ready for consistency analysis

### Step 2: Terminology Consistency Audit
**Action**: Check terminology usage against terminology.md across all files
**Input**: All files + terminology.md
**Output**: Terminology audit report

#### Audit Categories

1. **Phase Names**: Exact match to terminology.md definitions
   - Check: Every reference to a phase uses the exact name
   - Example: "SETUP PHASE" not "Setup Phase" or "setup phase" (match the convention)

2. **Stage Names**: Exact match to terminology.md definitions
   - Check: Every reference to a stage uses the exact name
   - Example: "Code Analysis" not "Code Analyzing" or "code analysis"

3. **Execution Classifications**: Consistent terminology
   - Check: ALWAYS, CONDITIONAL, not variations like "always", "Required", "Optional"
   - Check: Adaptive depth uses "Minimal/Standard/Comprehensive" consistently

4. **Agent Type Terms**: Consistent usage
   - Check: "target agent" not "the agent" or "your agent" inconsistently
   - Check: Agent type labels match (Process/Task/Analytical/Hybrid)

5. **Domain-Specific Terms**: Match glossary definitions
   - Check: All domain terms used as defined
   - Check: No undefined domain terms introduced in later files

6. **Action Verbs**: Consistent verb choices
   - Check: "Execute" vs "Run" vs "Perform" — pick one per context
   - Check: "Generate" vs "Create" vs "Produce" — pick one per context
   - Check: "Validate" vs "Verify" vs "Check" — pick one per context

```markdown
## Terminology Audit Results

### Phase Name Consistency
| Phase | Expected | Files with Violations | Count |
|-------|----------|----------------------|-------|
| [Phase 1] | [Exact Name] | [files] | [N] |

### Stage Name Consistency
[Similar table]

### Classification Consistency
[Similar table]

### Domain Term Consistency
[Similar table]

**Total Violations**: [N]
```

### Step 3: Structural Pattern Consistency Audit
**Action**: Check structural patterns across all files
**Input**: All files + output-structure-patterns.md
**Output**: Structural pattern audit report

#### Pattern Checks

1. **Heading Style**:
   - Consistent heading hierarchy (H1 for title, H2 for sections, H3 for subsections)
   - Consistent case convention for headings
   - Consistent separator usage between major sections

2. **Section Order**:
   - Phase rule files follow consistent section order
   - Common rule files follow consistent section order
   - All files have required sections in expected positions

3. **List Format**:
   - Consistent list markers (all `-` or all `*`)
   - Consistent indentation for nested lists
   - Consistent use of numbered vs bulleted lists

4. **Code Block Format**:
   - Consistent language identifiers
   - Consistent indentation within code blocks
   - Consistent use of inline code vs code blocks

5. **Table Format** (where applicable):
   - Consistent column headers
   - Consistent alignment
   - Consistent use of tables vs lists for similar content

### Step 4: Completion Message Format Consistency
**Action**: Check all completion messages follow the same format
**Input**: All phase rule files
**Output**: Completion message consistency report

For each completion message, verify:
- [ ] Uses "### REVIEW REQUIRED" heading
- [ ] Has summary of what was produced
- [ ] Uses "### WHAT'S NEXT" heading
- [ ] Has exactly 2 options
- [ ] Option A is "**A) Request Changes**" with description
- [ ] Option B is "**B) Continue to [Next Stage]**" with correct stage name
- [ ] Surrounded by `---` separators
- [ ] Bold formatting consistent

### Step 5: Question Format Consistency
**Action**: Verify question format instructions are consistent
**Input**: All files that reference question generation
**Output**: Question format consistency report

Check:
- All question references point to question-format-guide.md
- Question category descriptions are consistent
- Question file naming conventions are consistent
- Answer format references are consistent

### Step 6: Error Handling Format Consistency
**Action**: Verify error handling sections follow consistent patterns
**Input**: All files with error handling sections
**Output**: Error handling format consistency report

Check:
- Error descriptions follow consistent format
- Severity levels use same terms (Critical/High/Medium/Low)
- Recovery procedure format is consistent
- Escalation language is consistent
- Reference to common error-handling.md is present

### Step 7: Cross-Reference Format Consistency
**Action**: Verify cross-references use consistent format
**Input**: All files
**Output**: Cross-reference format consistency report

Check:
- File path format is consistent (relative paths, same style)
- Reference descriptions follow consistent pattern
- Link format is consistent (inline vs reference-style)
- "See [file] for..." phrasing is consistent

### Step 8: Writing Style Consistency
**Action**: Check writing style for consistency across all files
**Input**: All files
**Output**: Style consistency report

Check:
- **Voice**: Consistent use of imperative vs instructional
- **Tone**: Consistent level of formality
- **Person**: Consistent use of "you" vs "the AI" vs "the agent"
- **Emphasis**: Consistent use of bold, italic, and CAPS
- **MANDATORY/CRITICAL markers**: Used consistently and sparingly

### Step 9: Fix Identified Inconsistencies
**Action**: Fix inconsistencies found during the audit
**Input**: All audit reports
**Output**: Fixed files

**Fix Priority**:
1. **Terminology violations**: Critical — fix all
2. **Completion message format**: Critical — fix all
3. **Structural patterns**: High — fix most
4. **Writing style**: Medium — fix obvious ones
5. **Formatting details**: Low — fix if easy

**Fix Approach**:
- For terminology: Update to match terminology.md
- For patterns: Update to match output-structure-patterns.md
- For style: Standardize to the most common pattern used

### Step 10: Re-Validate After Fixes
**Action**: Re-run consistency checks after fixes
**Input**: Fixed files
**Output**: Updated audit reports

Verify:
- All critical fixes applied
- No new inconsistencies introduced by fixes
- Overall consistency improved

### Step 11: Generate Consistency Report
**Action**: Compile all audit results into final report
**Input**: All audit results (before and after fixes)
**Output**: Consistency review report

### Step 12: Present Consistency Report
**Action**: Present the consistency review for approval
**Input**: Consistency report
**Output**: Formatted report for user review

---

## Output Artifacts

### Consistency Review Report
- **File**: `steering-docs/refinement/consistency-review-report.md`
- **Content**: Complete audit results and fix summary
- **Format**:

```markdown
# Consistency Review Report

## Summary
| Category | Items Checked | Consistent | Violations | Fixed |
|----------|--------------|------------|------------|-------|
| Terminology | [N] | [N] | [N] | [N] |
| Structural Patterns | [N] | [N] | [N] | [N] |
| Completion Messages | [N] | [N] | [N] | [N] |
| Question Format | [N] | [N] | [N] | [N] |
| Error Handling | [N] | [N] | [N] | [N] |
| Cross-References | [N] | [N] | [N] | [N] |
| Writing Style | [N] | [N] | [N] | [N] |

## Overall Consistency Score: [N]%
## Status: [PASS / NEEDS WORK]

[Detailed sections for each category]

## Fixes Applied
[Summary of changes made]

## Remaining Issues
[Any issues not fixed, with justification]
```

---

## Domain Specificity Measurement (Dim 12)

### Measurement Procedure

For each Phase Rule file, measure domain specificity rate:

1. **Count total content lines**: Total lines - blank lines - heading lines
2. **Count domain-specific lines**: Lines containing:
   - Terms defined in terminology.md
   - Domain-specific proper nouns from Domain Research
   - Domain-specific procedures, checks, standards
   - NOT generic template boilerplate
3. **Calculate**: domain-specific lines ÷ total content lines
4. **Threshold**: >= 40% per file

### Domain Specificity Heatmap

```markdown
## Domain Specificity by File

| File | Total Lines | Domain Lines | Rate | Status |
|------|------------|-------------|------|--------|
| discovery/purpose-analysis.md | [N] | [N] | [N]% | [OK/LOW] |
| discovery/domain-research.md | [N] | [N] | [N]% | [OK/LOW] |
| ... | ... | ... | ... | ... |

**Average**: [N]% | **Below 40%**: [N] files
```

### 4-Agent-Type Consistency Check

Verify references to the 4 agent types (Process/Task/Analytical/Hybrid) are consistent:
- [ ] All phase rule files that mention agent types use all 4 types consistently
- [ ] No agent type is described differently across files
- [ ] Agent type characteristics match terminology.md definitions

### Handling Results

R2 is a **measurement and minor-fix stage**, not a repair-routing stage:
- **Minor issues** (terminology corrections, formatting): Fix in-place during R2
- **Structural issues** (domain specificity too low, missing sections): Measure and report. These feed into R3 Quality Calibration for repair routing via the judgment tree.

---

## Completion Message

### REVIEW REQUIRED

**Consistency Review is complete.** Here's the summary:

- **Total Items Checked**: [N]
- **Violations Found**: [N]
- **Violations Fixed**: [N] (minor, in-place)
- **Remaining Issues**: [N] (reported to R3)
- **Consistency Score**: [N]%
- **Domain Specificity**: Average [N]%, [N] files below 40%

Key findings:
- [Finding 1]
- [Finding 2]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll address additional consistency issues based on your feedback
**B) Continue to Quality Calibration** — Proceed to calibrate against AI-DLC quality standards

---

## Error Handling

### Widespread Inconsistencies
- **Issue**: Inconsistencies are pervasive across many files
- **Solution**: Identify root cause (usually terminology or early template issue), fix systematically
- **Recovery**: May need to regenerate files with consistent templates

### Fixes Introduce New Issues
- **Issue**: Fixing inconsistencies breaks other things (e.g., updating a term breaks references)
- **Solution**: Track all dependencies before making changes, update in correct order
- **Recovery**: Revert to pre-fix state, plan fixes more carefully

### Style Subjectivity
- **Issue**: Some style differences are preference-based rather than wrong
- **Solution**: Choose one consistent approach, document the convention
- **Note**: Consistency is more important than any specific style choice

## References
- `common/terminology.md` — Source of truth for terminology
- `common/output-structure-patterns.md` — Source of truth for patterns
- `refinement/completeness-review.md` — Prior review results
- `common/content-validation.md` — Content standards
