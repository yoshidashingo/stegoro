# Output Structure Patterns

## Purpose

This file defines the standardized structural patterns that ALL generated proposal artifacts must follow. These patterns ensure consistency across proposal documents and make outputs predictable for both the workflow and human reviewers.

---

## Proposal Artifact Patterns

### RFP Summary Pattern

```markdown
# RFP Summary — [RFP Title]

## RFP Overview
- **Issuing Organization**: [Name]
- **RFP Number**: [Number/ID]
- **Issue Date**: [Date]
- **Submission Deadline**: [Date and Time]
- **Q&A Deadline**: [Date, if applicable]
- **Contract Type**: [FFP / T&M / CPFF / etc.]
- **Estimated Value**: [Range or amount, if disclosed]

## Scope of Work
[Brief description of what is being procured]

## Document Structure
| Section | Title | Pages | Key Content |
|---------|-------|-------|-------------|
| [Num] | [Title] | [Pages] | [Summary] |

## Key Dates
| Event | Date |
|-------|------|
| [Event] | [Date] |

## Submission Requirements
- **Format**: [Electronic / Physical / Both]
- **Page Limit**: [Number, if applicable]
- **Copies**: [Number required]
- **Delivery Method**: [Email / Portal / Mail]

## Evaluation Methodology
[Summary of how proposals will be evaluated]
```

---

### Requirements Document Pattern

```markdown
# Requirements — [RFP Title]

## Requirements Summary
- **Total Requirements**: [N]
- **Mandatory**: [N]
- **Desirable**: [N]
- **Informational**: [N]

## Mandatory Requirements

### [Category 1]
| Req ID | Description | Source | Priority | Notes |
|--------|-------------|--------|----------|-------|
| [ID] | [Description] | [RFP Section] | Mandatory | [Notes] |

### [Category 2]
[Continue for all categories]

## Desirable Requirements
[Same format as above]

## Informational Requirements
[Same format as above]

## Evaluation Criteria
| Criterion | Weight | Description |
|-----------|--------|-------------|
| [Criterion] | [%] | [Description] |

## Ambiguities and Clarification Needs
| Item | Description | Recommended Action |
|------|-------------|--------------------|
| [Item] | [Description] | [Action] |
```

---

### Compliance Matrix Pattern

```markdown
# Compliance Matrix — [RFP Title]

## Compliance Summary
- **Total Requirements**: [N]
- **Compliant**: [N] ([%])
- **Partially Compliant**: [N] ([%])
- **Non-Compliant**: [N] ([%])
- **Not Applicable**: [N] ([%])

## Detailed Compliance Matrix

| Req ID | Requirement | Status | Proposal Section | Notes |
|--------|-------------|--------|-----------------|-------|
| [ID] | [Brief description] | Compliant / Partial / Non-Compliant / N/A | [Section ref] | [Mitigation or notes] |

## Risk Areas
[Requirements with Partial or Non-Compliant status that carry high evaluation weight]

## Mitigation Strategies
[For each Partial/Non-Compliant item: approach to minimize impact]
```

---

### Proposal Section Pattern

```markdown
# [Section Title]

## [Subsection 1 Title]
[Content addressing specific RFP requirements]

**Requirement Addressed**: [Req ID(s)]

[Detailed response content]

## [Subsection 2 Title]
[Continue for all subsections]

---

**Requirements Addressed in This Section**:
| Req ID | Status | Description |
|--------|--------|-------------|
| [ID] | Compliant | [Brief note] |
```

---

### Submission Checklist Pattern

```markdown
# Submission Checklist — [RFP Title]

## Pre-Submission Verification

### Document Completeness
- [ ] Executive Summary included
- [ ] Technical Response complete
- [ ] Management Response included (if required)
- [ ] Pricing Framework included (if required)
- [ ] All appendices attached
- [ ] Compliance matrix included

### Format Compliance
- [ ] Page limit verified: [X] pages (limit: [Y])
- [ ] Font and formatting per RFP requirements
- [ ] Headers/footers as specified
- [ ] Page numbering correct
- [ ] Table of contents accurate

### Content Verification
- [ ] All mandatory requirements addressed
- [ ] Compliance matrix complete
- [ ] Cross-references verified
- [ ] No placeholder text remaining
- [ ] Company name and branding consistent

### Submission Logistics
- [ ] Submission format confirmed: [Electronic/Physical]
- [ ] Delivery method: [Method]
- [ ] Deadline: [Date and Time]
- [ ] Recipient: [Name/Address]
- [ ] Copies: [Number]

## Final Approval
- [ ] Authorized signatory reviewed
- [ ] Legal review complete (if required)
- [ ] Pricing approved by authorized personnel
```

---

## State and Audit File Patterns

### State File Pattern

See `common/session-continuity.md` for the complete state file format.

### Audit File Pattern

```markdown
# Proposal Writer — Audit Log

## [Stage Name or Interaction Type]
**Timestamp**: [ISO 8601 timestamp]
**User Input**: "[Complete raw user input — never summarized]"
**AI Response**: "[AI's response or action taken]"
**Context**: [Stage, action, or decision made]

---

[Continue for each interaction]
```

---

## Completion Message Patterns

### Stage Completion (Standard 2-Option)

```markdown
---

### REVIEW REQUIRED

**[Stage Name] is complete.** Here's a summary of what was produced:

- [Output 1 description]
- [Output 2 description]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise based on your feedback
**B) Continue to [Next Stage Name]** — Proceed to the next stage

---
```

### Phase Completion

```markdown
---

### PHASE COMPLETE: [PHASE NAME]

**All stages in the [Phase Name] phase are complete.**

**Summary of outputs:**
- [Key output 1]
- [Key output 2]

### WHAT'S NEXT

The next phase is **[Next Phase Name]**: [Brief description].

**A) Proceed to [Next Phase Name]** — Continue the workflow
**B) Review [Current Phase Name] outputs** — Review before moving forward

---
```

### Workflow Completion

```markdown
---

### PROPOSAL COMPLETE

**The proposal for [RFP Title] has been generated and reviewed.**

**Final Deliverables:**
- Directory: `proposal-docs/`
- Compliance score: [X]% requirements addressed
- Quality score: [X]/22 quality dimensions

**Contents:**
- Executive Summary
- Technical Response
- [Additional sections as applicable]
- Compliance Matrix
- Submission Checklist

### NEXT STEPS

1. Review the complete proposal in `proposal-docs/`
2. Conduct internal review with subject matter experts
3. Prepare final submission package per RFP instructions
4. Submit before the deadline

---
```

---

## Implementation Guidelines

1. **Follow patterns exactly**: Do not deviate from established patterns without justification
2. **Include all fields**: Even optional fields should be present (marked as N/A if not applicable)
3. **Maintain consistency**: Use the same pattern format across all instances
4. **Cross-reference accurately**: All section references must resolve to actual content

## References

- `common/content-validation.md` — Validation rules for generated content
- `common/terminology.md` — Standardized terminology
- `common/quality-standards.md` — Quality dimension requirements
