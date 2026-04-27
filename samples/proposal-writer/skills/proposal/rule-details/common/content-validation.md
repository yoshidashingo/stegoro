# Content Validation Rules

## Purpose

This file defines the validation rules that MUST be applied before creating ANY file during the proposal creation workflow. Content validation prevents formatting errors, broken references, and inconsistencies that undermine proposal quality.

---

## Pre-Creation Validation Checklist

Before writing any proposal artifact, verify:

### 1. Document Structure Validation
- [ ] All headings follow correct hierarchy (H1 > H2 > H3, no skipping levels)
- [ ] Table of contents entries match actual section headings
- [ ] Section numbering is sequential and consistent
- [ ] No orphaned sections (sections without content)

### 2. Cross-Reference Validation
- [ ] All internal cross-references point to existing sections
- [ ] Compliance matrix references map to actual proposal sections
- [ ] Requirement IDs in proposal match requirement IDs in RFP analysis
- [ ] Figure and table numbers are sequential and referenced in text

### 3. Formatting Validation
- [ ] Consistent heading capitalization style throughout
- [ ] Consistent list formatting (bullets vs. numbers) within sections
- [ ] Tables have headers and consistent column alignment
- [ ] No mixed indentation styles within the same file

### 4. Content Integrity Validation
- [ ] No placeholder text remaining (e.g., "[TBD]", "[Insert here]")
- [ ] No contradictory statements between sections
- [ ] Technical terms used consistently throughout
- [ ] Acronyms defined on first use in each major section

### 5. Special Character Handling
- [ ] Markdown special characters properly escaped when used literally
- [ ] Currency symbols and numbers formatted consistently
- [ ] Date formats consistent (ISO 8601 preferred for internal docs)
- [ ] Percentage and unit formats consistent

---

## Proposal-Specific Validation Rules

### Compliance Matrix Validation
- Every RFP requirement ID must appear in the compliance matrix
- Every compliance matrix entry must have a status (Compliant / Partial / Non-Compliant / N/A)
- Partial and Non-Compliant entries must include mitigation notes
- Matrix must cross-reference to the proposal section where each requirement is addressed

### Requirements Traceability Validation
- Every extracted requirement must trace to a source location in the RFP
- Every proposal section must trace back to at least one RFP requirement
- No proposal section should exist without corresponding RFP justification
- Forward and backward traceability must be complete

### Section Length Validation
- If RFP specifies page limits, track estimated page counts per section
- Flag sections exceeding allocated page budget
- Ensure critical sections receive adequate page allocation
- Track total page count against overall limit

### Consistency Validation Across Sections
- Company name and branding must be identical across all sections
- Team member names and titles must be consistent
- Technical terms and product names must match throughout
- Project timeline dates must be consistent across sections

---

## Validation Process

### When to Validate
1. **Before creating** any new file
2. **After completing** each proposal section
3. **During compliance check** in the Review phase
4. **Before final assembly** of the complete proposal

### Validation Severity Levels

**BLOCKING** — Must fix before proceeding:
- Broken cross-references
- Missing mandatory sections
- Compliance matrix gaps for mandatory requirements
- Contradictory content between sections

**WARNING** — Should fix, but can proceed with acknowledgment:
- Minor formatting inconsistencies
- Non-mandatory section gaps
- Estimated page budget overruns (within 10%)

**INFO** — Good to fix, does not block progress:
- Style preferences (e.g., Oxford comma usage)
- Optional section enhancements
- Cosmetic formatting variations

---

## Implementation Guidelines

1. **Validate incrementally**: Check each section as it's written, not just at the end
2. **Log validation results**: Record validation outcomes in audit.md
3. **Fix blocking issues immediately**: Do not proceed with blocking validation failures
4. **Aggregate warnings**: Present all warnings at phase completion for batch resolution
5. **Maintain a running compliance score**: Track percentage of requirements addressed

## Error Handling

- If validation reveals contradictions: Flag and present to user for resolution before proceeding
- If cross-references are broken: Identify source and target, suggest correction
- If page limits are exceeded: Present options for content reduction with impact assessment
- See `common/error-handling.md` for general error handling procedures

## References

- `common/quality-standards.md` — Quality dimension scoring framework
- `common/terminology.md` — Standardized terminology for consistency checks
- `common/output-structure-patterns.md` — Expected file structure patterns
