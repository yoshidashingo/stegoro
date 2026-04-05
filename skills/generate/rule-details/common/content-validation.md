# Content Validation Rules

## MANDATORY: Content Validation Before File Creation

**CRITICAL**: All generated content MUST be validated before writing to files to prevent parsing errors. This applies to both the Steering Policy Maker's own working artifacts AND the generated policy files for target agents.

## Two-Level Validation

### Level 1: Working Artifact Validation
Validation of files created during the steering policy creation process (question files, plans, reports).

### Level 2: Generated Policy File Validation
Validation of the final policy files generated for the target agent. This level requires stricter validation because these files will be used by the target agent in production.

---

## ASCII Diagram Standards

**CRITICAL**: Before creating ANY file with ASCII diagrams:

1. **VALIDATE** each diagram:
   - Count characters per line (all lines MUST be same width within the diagram)
   - Use ONLY: `+` `-` `|` `^` `v` `<` `>` and spaces
   - NO Unicode box-drawing characters
   - Spaces only (NO tabs)
2. **TEST** alignment by verifying box corners align vertically
3. **PROVIDE** text alternative alongside every diagram

### ASCII Diagram Pattern

```
+------------------+     +------------------+     +------------------+
|   Phase 1        |---->|   Phase 2        |---->|   Phase 3        |
|   DISCOVERY      |     |   DESIGN         |     |   GENERATION     |
+------------------+     +------------------+     +------------------+
                                                           |
                          +------------------+             v
                          |   Phase 5        |     +------------------+
                          |   PACKAGING      |<----|   Phase 4        |
                          +------------------+     |   REFINEMENT     |
                                                   +------------------+
```

---

## Mermaid Diagram Validation

### Required Validation Steps
1. **Syntax Check**: Validate Mermaid syntax before file creation
2. **Character Escaping**: Ensure special characters are properly escaped
3. **Fallback Content**: Provide text alternative if Mermaid fails validation

### Mermaid Validation Rules

Before creating any file with Mermaid diagrams:

1. Check for invalid characters in node IDs (use alphanumeric + underscore only)
2. Escape special characters in labels: `"` to `\"` and `'` to `\'`
3. Validate flowchart syntax: node connections must be valid
4. Test diagram parsing with simple validation
5. Ensure no unterminated strings or brackets

### Implementation Pattern

```markdown
## Workflow Visualization

### Mermaid Diagram (if syntax valid)
` ` `mermaid
[validated diagram content]
` ` `

### Text Alternative (always include)
` ` `
Phase 1: DISCOVERY
  - Stage 1: Purpose Analysis (COMPLETED)
  - Stage 2: Domain Research (COMPLETED)
[continue with text representation]
` ` `
```

**Note**: Always include the text alternative, even if the Mermaid diagram validates successfully. This ensures content is accessible in all rendering contexts.

---

## Markdown Syntax Validation

### Heading Validation
- Verify heading hierarchy (no skipped levels: H1 → H3 without H2)
- Ensure consistent heading style (ATX `#` headers, not Setext underlines)
- Check for proper spacing after `#` characters

### Link Validation
- Verify all internal links reference existing files or sections
- Use relative paths for file references within the policy set
- Check for broken anchor links to headings

### List Validation
- Consistent list markers within the same list (all `-` or all `*`)
- Proper indentation for nested lists
- No mixed ordered/unordered within same level

### Code Block Validation
- All code blocks properly opened and closed
- Language identifiers specified where applicable
- No nested code blocks (which cause parsing issues)

---

## Cross-Reference Validation

### File Reference Rules
When a generated policy file references another file:

1. **Verify the referenced file exists** (or will be created in the current batch)
2. **Use consistent path format**: relative paths from the file's location
3. **Include both path and description**: `common/terminology.md` (Terminology Glossary)

### Cross-Reference Pattern

```markdown
## MANDATORY: Load Related Rules
- Load `common/question-format-guide.md` for question formatting
- Load `common/content-validation.md` for validation requirements
- Reference `common/terminology.md` for term definitions
```

### Validation Checklist for Cross-References
- [ ] All referenced files exist in the generated policy set
- [ ] All file paths are correct and relative
- [ ] No circular references that could cause infinite loading
- [ ] Referenced sections/headings exist in target files

---

## Generated Content Validation for Target Agent Policies

### Pre-Creation Validation Checklist
Before writing any generated policy file:

- [ ] Validate embedded code blocks (Mermaid, JSON, YAML, markdown examples)
- [ ] Check special character escaping
- [ ] Verify markdown syntax correctness
- [ ] Test content parsing compatibility
- [ ] Include fallback content for complex visual elements
- [ ] Verify all cross-references resolve to actual files
- [ ] Check terminology consistency against terminology.md
- [ ] Verify ALWAYS/CONDITIONAL classifications are explicit
- [ ] Confirm completion message format follows standard pattern
- [ ] Ensure question formats follow question-format-guide.md

### Content Structure Validation
For each generated rule file, verify:

1. **Header section**: Clear title, purpose statement
2. **Prerequisites section**: What must be loaded/completed first
3. **Steps section**: Numbered steps with clear instructions
4. **Checkpoint section**: Approval gate with standardized format
5. **Completion message section**: REVIEW REQUIRED + WHAT'S NEXT format
6. **Error handling section**: What to do when things go wrong

---

## Error Prevention Rules

1. **Always validate before writing files**: Never write unvalidated content
2. **Escape special characters**: Particularly in diagrams and code blocks
3. **Provide alternatives**: Include text versions of visual content
4. **Test syntax**: Validate complex content structures
5. **Incremental writing**: Write and validate one file at a time rather than batching

---

## Validation Failure Handling

### When Validation Fails

1. **Log the error**: Record what failed validation in audit.md
2. **Identify the issue**: Determine exactly which content element failed
3. **Fix in-place**: Correct the issue before writing the file
4. **Re-validate**: Run validation again after fixes
5. **Use fallback content**: If fix is not possible, switch to text-based alternative
6. **Continue workflow**: Don't block the entire workflow on a single validation failure
7. **Inform user**: Mention simplified content was used due to parsing constraints

### Validation Error Severity

**Blocking**: Issues that prevent the file from being usable
- Broken markdown structure
- Invalid cross-references to non-existent files
- Missing required sections (steps, checkpoints)

**Non-Blocking**: Issues that degrade quality but don't prevent use
- Mermaid diagram syntax errors (use text fallback)
- Minor formatting inconsistencies
- Optional sections missing

---

## Plugin Structure Validation

### Required Checks for PACKAGING Phase

When generating plugin structure (P1/P2), validate:

1. **plugin.json**: Required fields present (name, version, description, agents, skills, commands)
2. **SKILL.md line count**: 75-150 lines per skill entry point
3. **core-workflow line count**: 200-600 lines
4. **Phase Rule file line count**: 100-300 lines each
5. **Common Rule file line count**: 50-200 lines each
6. **Directory structure**: All referenced files exist in the plugin tree

### Content Density Guard

Based on Domain Research Pattern 6 (content density limits):
- Files exceeding upper line limit → split or delegate to sub-files
- Files below lower line limit → insufficient depth, add domain content

### Self-Application Exclusion

**The line count guidelines are designed for target agent's generated files, not for the SPM's own rule-details.** The SPM (meta-agent, Process Agent, Comprehensive depth) is inherently more detailed than the policies it generates. SPM's own common rules and phase rules are the generator's instructions and naturally exceed the guidelines intended for generated output. This exclusion applies only to SPM's own files; generated target agent files must comply with the guidelines.

---

## Dim 12-15 Pre-Validation (GENERATION Phase)

During Phase Rules Generation (G3), perform early quality checks before Integration Validation:

| Dimension | Pre-Check | Action if Below Threshold |
|-----------|-----------|--------------------------|
| Dim 12: Domain Specificity Rate | Count terminology.md terms in each Phase Rule file ÷ total lines | Flag for additional domain content injection |
| Dim 13: Example Coverage | Count GOOD/BAD example blocks per file | Add examples from Domain Research best practices/pitfalls |
| Dim 14: Artifact Template Completeness | Check output-producing files for template blocks | Add template sections before writing file |
| Dim 15: Pitfall Reference Rate | Check error handling items for domain-research citations | Add pitfall-derived error scenarios |

**Note**: These are early warnings, not final measurements. Final scoring occurs in R3 Quality Calibration.

---

## Batch Validation (Integration Validation Stage)

During the GENERATION phase's Integration Validation stage, perform comprehensive cross-file validation:

1. **Reference Graph**: Build a graph of all file references and verify completeness
2. **Terminology Audit**: Check all files use terms consistently per terminology.md
3. **Pattern Compliance**: Verify all files follow structural patterns from output-structure-patterns.md
4. **Flow Validation**: Trace every possible workflow path and verify all stages have corresponding rule files
5. **Coverage Check**: Verify all ALWAYS stages have fully detailed rule files

### Batch Validation Report Format

```markdown
# Integration Validation Report

## File Reference Validation
- Total references: [N]
- Resolved: [N]
- Broken: [N] (list each)

## Terminology Consistency
- Terms checked: [N]
- Consistent: [N]
- Inconsistent: [N] (list each with file and line)

## Pattern Compliance
- Files checked: [N]
- Compliant: [N]
- Non-compliant: [N] (list each with issues)

## Flow Validation
- Workflow paths traced: [N]
- Complete: [N]
- Incomplete: [N] (list each with missing stages)

## Overall Status: [PASS / FAIL]
```
