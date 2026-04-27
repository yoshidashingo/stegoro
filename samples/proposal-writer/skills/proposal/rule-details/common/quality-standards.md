# Quality Standards — 11 Quality Dimensions

## Purpose

This file defines the quality framework used to evaluate the proposal creation workflow and its outputs. Every workflow execution is measured against 11 quality dimensions to ensure consistent, professional, compliant proposals.

---

## The 11 Quality Dimensions

### Dimension 1: Adaptive Workflow
**What**: All stages are explicitly classified as ALWAYS or CONDITIONAL with clear criteria.
**Why**: Prevents wasted effort on unnecessary sections while ensuring mandatory components are never skipped.

**PASS**: Every stage has a clear classification. CONDITIONAL stages have explicit Execute IF / Skip IF criteria. Stage decisions are logged.
**PARTIAL**: Classifications exist but some CONDITIONAL criteria are vague or incomplete.
**FAIL**: Stages lack classification or CONDITIONAL stages execute/skip without documented criteria.

### Dimension 2: Mandatory Checkpoints
**What**: Critical decisions require explicit user approval with standardized 2-option messages.
**Why**: Ensures users maintain control over strategic decisions and content direction.

**PASS**: Every stage ends with "REVIEW REQUIRED" + "WHAT'S NEXT" (exactly 2 options: Request Changes / Continue). User approval is logged before proceeding.
**PARTIAL**: Most stages have checkpoints but some use non-standard formats or more than 2 options.
**FAIL**: Stages proceed without user approval or use inconsistent checkpoint formats.

### Dimension 3: Question File Format
**What**: ALL questions use dedicated files with multiple-choice format + "Other" option + [Answer]: tags.
**Why**: Structured questions ensure clear communication and enable automated validation.

**PASS**: All questions follow the template in `question-format-guide.md`. Contradictions are detected and clarification questions generated.
**PARTIAL**: Questions are structured but some deviate from the template or miss the "Other" option.
**FAIL**: Questions are asked in chat instead of dedicated files, or lack structured format.

### Dimension 4: Content Validation
**What**: Pre-creation validation rules for document structure, cross-references, and formatting.
**Why**: Prevents errors that undermine proposal credibility and compliance.

**PASS**: Every file is validated before creation. Cross-references verified. Compliance matrix entries all resolve correctly.
**PARTIAL**: Validation occurs but some categories are skipped or inconsistently applied.
**FAIL**: Files are created without validation, or validation results are ignored.

### Dimension 5: Audit Trail
**What**: ISO 8601 timestamps, complete raw user inputs (never summarized), append-only logging.
**Why**: Enables session resumption, decision traceability, and accountability.

**PASS**: Every interaction logged with timestamp. User inputs captured verbatim. audit.md is append-only (never overwritten).
**PARTIAL**: Most interactions logged but some gaps exist or formatting is inconsistent.
**FAIL**: Audit logging is incomplete, user inputs are summarized, or audit.md is overwritten.

### Dimension 6: Error Handling
**What**: Phase-specific recovery procedures for all severity levels.
**Why**: Ensures the workflow can recover gracefully without losing work.

**PASS**: All error types covered. Recovery procedures exist for each phase. Errors logged in audit.md. Users informed of errors and options.
**PARTIAL**: Common errors handled but edge cases lack procedures.
**FAIL**: Errors cause workflow failure without recovery or communication.

### Dimension 7: Overconfidence Prevention
**What**: "When in doubt, ask" philosophy with stage-specific guidance.
**Why**: Prevents incorrect assumptions that lead to non-compliant or off-target proposals.

**PASS**: Red flags trigger questions. Assumptions are documented. Company-specific claims verified with user. No unverified assertions in proposal.
**PARTIAL**: Most assumptions caught but some areas rely on unverified defaults.
**FAIL**: Proposal contains unverified claims, assumed capabilities, or unsupported assertions.

### Dimension 8: Depth Levels
**What**: Adaptive detail (Minimal / Standard / Comprehensive) based on RFP complexity.
**Why**: Ensures effort matches RFP complexity without under- or over-engineering.

**PASS**: Depth level assessed at workflow start. Each stage adapts execution to depth level. Simple RFPs are efficient; complex RFPs are thorough.
**PARTIAL**: Depth level set but not consistently applied across all stages.
**FAIL**: Same depth applied regardless of RFP complexity.

### Dimension 9: Session Continuity
**What**: State file tracking, context loading, session resumption with artifact validation.
**Why**: Enables multi-session workflows without losing progress.

**PASS**: proposal-state.md accurately tracks progress. Resumption validates all artifacts. Context is loaded correctly from prior sessions.
**PARTIAL**: State tracking exists but artifact validation is incomplete.
**FAIL**: Session resumption fails or loses context.

### Dimension 10: Terminology Standardization
**What**: Comprehensive glossary enforced across all files.
**Why**: Ensures consistent, professional language throughout the proposal.

**PASS**: RFP terminology used consistently. Glossary terms applied across all sections. Acronyms defined on first use.
**PARTIAL**: Most terms consistent but some variations exist between sections.
**FAIL**: Inconsistent terminology between proposal sections.

### Dimension 11: Standardized Completion Messages
**What**: Every stage ends with REVIEW REQUIRED + WHAT'S NEXT (exactly 2 options).
**Why**: Provides predictable user experience and prevents emergent navigation patterns.

**PASS**: Every stage uses the exact 2-option format. No 3-option menus. No emergent navigation patterns.
**PARTIAL**: Most stages follow the pattern but some deviate.
**FAIL**: Stages use inconsistent completion messages or more than 2 options.

---

## Quality Scoring

### Scoring Method
Each dimension is scored independently:
- **PASS** (2 points): Fully meets the dimension criteria
- **PARTIAL** (1 point): Partially meets criteria with minor gaps
- **FAIL** (0 points): Does not meet criteria

### Scoring Thresholds
- **Excellent**: 22/22 (all PASS)
- **Acceptable**: 18-21/22 (no FAIL scores)
- **Needs Improvement**: 14-17/22 (some FAIL scores)
- **Unacceptable**: Below 14/22 (multiple FAIL scores)

### Minimum Requirements
- **ALL 11 dimensions must score PASS or PARTIAL** — no FAIL scores allowed
- Dimensions 1, 2, 4, 5, 7 are critical — PASS required for these

---

## Quality Scorecard Template

```markdown
# Proposal Writer — Quality Scorecard

| # | Dimension | Score | Notes |
|---|-----------|-------|-------|
| 1 | Adaptive Workflow | PASS/PARTIAL/FAIL | [Details] |
| 2 | Mandatory Checkpoints | PASS/PARTIAL/FAIL | [Details] |
| 3 | Question File Format | PASS/PARTIAL/FAIL | [Details] |
| 4 | Content Validation | PASS/PARTIAL/FAIL | [Details] |
| 5 | Audit Trail | PASS/PARTIAL/FAIL | [Details] |
| 6 | Error Handling | PASS/PARTIAL/FAIL | [Details] |
| 7 | Overconfidence Prevention | PASS/PARTIAL/FAIL | [Details] |
| 8 | Depth Levels | PASS/PARTIAL/FAIL | [Details] |
| 9 | Session Continuity | PASS/PARTIAL/FAIL | [Details] |
| 10 | Terminology Standardization | PASS/PARTIAL/FAIL | [Details] |
| 11 | Standardized Completion Messages | PASS/PARTIAL/FAIL | [Details] |

**Total Score**: [X]/22
**Overall Rating**: [Excellent/Acceptable/Needs Improvement/Unacceptable]
```

---

## Implementation Guidelines

1. **Evaluate continuously**: Don't wait until the end — check quality at each phase
2. **Fix issues immediately**: Address FAIL scores before proceeding to the next phase
3. **Log quality findings**: Record quality observations in audit.md
4. **Use the scorecard**: Generate a quality scorecard at workflow completion

## References

- `common/content-validation.md` — Content validation procedures (Dimension 4)
- `common/error-handling.md` — Error handling procedures (Dimension 6)
- `common/overconfidence-prevention.md` — Assumption management (Dimension 7)
- `common/session-continuity.md` — State tracking procedures (Dimension 9)
- `common/terminology.md` — Terminology standards (Dimension 10)
- `common/output-structure-patterns.md` — File structure patterns
