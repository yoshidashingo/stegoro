# Error Handling and Recovery Procedures

## Purpose

This file defines error handling procedures for all phases of the proposal creation workflow. Proper error handling ensures the workflow can recover gracefully from failures without losing work or producing non-compliant proposals.

---

## General Error Handling Principles

### When Errors Occur
1. **Identify the error**: Clearly state what went wrong
2. **Assess impact**: Determine if the error is blocking or can be worked around
3. **Communicate**: Inform the user about the error and options
4. **Offer solutions**: Provide clear steps to resolve or work around the error
5. **Document**: Log the error and resolution in `audit.md`

### Error Severity Levels

**Critical**: Workflow cannot continue
- RFP document cannot be read or parsed
- Required proposal sections cannot be generated
- System errors preventing file operations

**High**: Phase cannot complete as planned
- Incomplete answers to required questions
- Contradictory user responses
- Missing dependencies from prior phases
- Compliance matrix reveals unresolvable gaps

**Medium**: Phase can continue with workarounds
- Optional artifacts missing
- Non-critical validation failures
- Partial completion possible
- Minor formatting issues

**Low**: Minor issues that don't block progress
- Style inconsistencies
- Optional information missing
- Non-blocking warnings

---

## Phase-Specific Error Handling

### Analysis Phase Errors

**Error**: RFP document is unreadable or corrupted
- **Cause**: Unsupported format, encoding issues, corrupted file
- **Solution**: Ask user to provide RFP in a supported format (PDF, DOCX, Markdown, plain text)
- **Workaround**: Work with user's verbal description of RFP requirements

**Error**: RFP structure is unclear or non-standard
- **Cause**: Informal RFP, email-based request, no structured sections
- **Solution**: Ask clarifying questions to structure the requirements
- **Workaround**: Create requirements from user-provided context

**Error**: Contradictory requirements in RFP
- **Cause**: Poorly written RFP, multiple authors, versioning issues
- **Solution**: Flag contradictions, ask user to clarify with the issuing organization
- **Do Not Proceed**: Until contradictions are documented and resolution approach is agreed

**Error**: Cannot determine mandatory vs. optional requirements
- **Cause**: RFP language is ambiguous (e.g., "should" vs. "shall" vs. "may")
- **Solution**: Ask user to classify ambiguous requirements
- **Default**: Treat ambiguous requirements as mandatory for compliance safety

### Planning Phase Errors

**Error**: Cannot design proposal structure within page limits
- **Cause**: Too many requirements for the allowed page count
- **Solution**: Present trade-off options (prioritize high-weight evaluation areas)
- **Workaround**: Propose condensed format with appendices for supplementary detail

**Error**: Win strategy cannot identify differentiators
- **Cause**: User cannot provide competitive intelligence
- **Solution**: Focus on customer-centric value propositions instead of competitive positioning
- **Workaround**: Skip ghost strategy, focus on strengths-based differentiation

**Error**: Content plan has sections without source material
- **Cause**: User lacks experience, past performance, or technical capability in an area
- **Solution**: Identify gaps, suggest teaming partners or mitigation strategies
- **Workaround**: Frame as development capability with credible plan

### Writing Phase Errors

**Error**: Executive summary exceeds allocated length
- **Cause**: Too many themes or requirements to address in limited space
- **Solution**: Prioritize key win themes and highest-weight evaluation criteria
- **Workaround**: Move detailed content to technical response, keep summary high-level

**Error**: Technical approach lacks specific detail
- **Cause**: Insufficient input from user on methodology, tools, or approach
- **Solution**: Ask targeted questions about technical approach
- **Do Not Proceed**: Until enough technical detail is provided for a credible response

**Error**: Management response references unavailable team members
- **Cause**: User-provided team information is incomplete or outdated
- **Solution**: Ask user to confirm team availability and credentials
- **Workaround**: Use role-based descriptions with "to be assigned" for specific individuals

**Error**: Pricing framework has inconsistent totals
- **Cause**: Calculation errors, missing line items, rate discrepancies
- **Solution**: Reconcile pricing elements, ask user to verify rates and quantities
- **Do Not Proceed**: Until pricing is internally consistent

### Review Phase Errors

**Error**: Compliance check reveals unaddressed mandatory requirements
- **Cause**: Requirement missed during extraction, section omitted during writing
- **Solution**: Return to Writing phase to address the gap
- **Recovery**: Add missing content to appropriate section, update compliance matrix

**Error**: Quality review finds contradictions between sections
- **Cause**: Sections written independently without cross-checking
- **Solution**: Present contradictions to user, reconcile content
- **Recovery**: Update affected sections to maintain consistency

**Error**: Final assembly cannot meet submission format requirements
- **Cause**: Formatting constraints not checked earlier, tool limitations
- **Solution**: Present format issues and ask user to prioritize corrections
- **Workaround**: Provide content in closest achievable format with notes on manual adjustments needed

---

## Recovery Procedures

### Partial Stage Completion

**Scenario**: Stage was interrupted mid-execution

**Recovery Steps**:
1. Load the stage plan file (if exists)
2. Identify last completed step (last [x] checkbox)
3. Resume from next uncompleted step
4. Verify all prior steps are actually complete
5. Continue execution normally

### Missing Artifacts

**Scenario**: Required artifacts from prior phase are missing

**Recovery Steps**:
1. Identify which artifacts are missing
2. Determine if they can be regenerated
3. If yes: Return to that phase, regenerate artifacts
4. If no: Ask user to provide information manually
5. Document the gap in `audit.md`

### User Wants to Restart Stage

**Scenario**: User is unhappy with stage results and wants to redo

**Recovery Steps**:
1. Confirm user wants to restart (current output will be archived)
2. Archive existing artifacts: `{artifact}.backup`
3. Reset stage status in `proposal-state.md`
4. Re-execute stage from beginning

### User Wants to Skip Stage

**Scenario**: User wants to skip a stage that was planned

**Recovery Steps**:
1. Confirm user understands implications
2. Document skip reason in `audit.md`
3. Mark stage as "SKIPPED" in `proposal-state.md`
4. Proceed to next stage
5. Note: May cause issues in later phases if dependencies missing

---

## Escalation Guidelines

### When to Ask for User Help

**Immediately**:
- Contradictory or ambiguous user input
- Missing required information about company capabilities
- Decisions requiring business judgment (pricing, teaming)
- RFP interpretation questions

**After Attempting Resolution**:
- Repeated errors in same step
- Complex technical requirements beyond available information
- Cross-section consistency issues
- Formatting challenges with available tools

---

## Logging Requirements

### Error Logging Format

```markdown
## Error - [Phase/Stage Name]
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
## Recovery - [Phase/Stage Name]
**Timestamp**: [ISO timestamp]
**Issue**: [What needed recovery]
**Recovery Steps**: [What was done]
**Outcome**: [Result of recovery]
**Artifacts Affected**: [List of files]

---
```

---

## Prevention Best Practices

1. **Validate early**: Check RFP completeness in the Analysis phase
2. **Checkpoint often**: Update state and checkboxes immediately after completing steps
3. **Communicate clearly**: Explain what you're doing and why at each stage
4. **Ask questions**: Don't assume — clarify ambiguities immediately
5. **Document everything**: Log all decisions and changes in `audit.md`
6. **Cross-check continuously**: Verify consistency between sections during writing
