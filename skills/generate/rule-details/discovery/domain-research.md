# Domain Research — DISCOVERY Phase

## Purpose
Research the target agent's domain to identify best practices, quality standards, common pitfalls, and domain-specific terminology. This ensures the generated steering policies incorporate real-world domain knowledge rather than generic patterns.

## Prerequisites
- Purpose Analysis completed and approved
- Agent type classification confirmed
- Common rules loaded

## Execution Classification
**Type**: ALWAYS (Adaptive depth)
This stage always executes but depth varies based on domain familiarity and complexity.

## Adaptive Depth

### MCP Tool Integration

When conducting domain research, use available MCP tools in this priority order:
1. **Context7** → Official documentation lookup (library/framework docs)
2. **Exa** → Web research for best practices, patterns, real-world examples
3. **gh search** → Existing implementation examples on GitHub
4. **Fallback**: If MCP tools are unavailable, ask user directly for domain knowledge, references, and standards

### Minimal Depth
**When**: Well-known domain with clear, established best practices
**Examples**: Code review, linting, simple documentation generation
**Scope**: Core best practices, major pitfalls only
**Minimum Research Items**: 3 (best practices >=2, pitfalls >=1)
**Expected Output**: ~1 page summary

### Standard Depth
**When**: Domain with some nuance requiring targeted research
**Examples**: Security auditing, API design review, test strategy planning
**Scope**: Best practices + standards + common pitfalls + quality criteria
**Minimum Research Items**: 7 (best practices >=3, pitfalls >=2, standards >=2)
**Expected Output**: ~2-3 page summary

### Comprehensive Depth
**When**: Novel, specialized, or high-risk domain
**Examples**: Medical device compliance, financial regulation auditing, safety-critical systems
**Scope**: Full research covering all categories with references and evidence
**Minimum Research Items**: 12 (best practices >=5, pitfalls >=3, standards >=2, design patterns >=2)
**Expected Output**: ~4-5 page summary with references

---

## Execution Steps

### Step 1: Determine Research Depth
**Action**: Assess the domain and select appropriate depth level
**Input**: Purpose analysis summary (agent type, domain, complexity)
**Output**: Depth level selection with justification

#### Depth Selection Criteria

| Factor | Minimal | Standard | Comprehensive |
|--------|---------|----------|---------------|
| Domain familiarity | Well-known | Moderate | Specialized |
| Regulatory requirements | None | Some | Extensive |
| Risk of incorrect agent behavior | Low | Medium | High |
| Established standards | Many, clear | Some | Few or complex |
| Domain terminology | Common | Some specialized | Highly specialized |
| User's domain expertise expectation | Basic | Moderate | Expert-level |

### Step 2: Research Domain Best Practices
**Action**: Identify established best practices for the domain
**Input**: Domain identification from purpose analysis
**Output**: Best practices catalog

Research areas:
1. **Industry Standards**: What standards or frameworks govern this domain?
2. **Process Patterns**: What are the accepted workflow patterns?
3. **Quality Metrics**: How is quality measured in this domain?
4. **Tool Ecosystem**: What tools and technologies are commonly used?
5. **Maturity Models**: Are there maturity models for this domain?

### Step 3: Identify Quality Standards and Compliance
**Action**: Catalog quality standards and compliance requirements
**Input**: Domain best practices
**Output**: Quality standards list

Consider:
- **Formal Standards**: ISO, IEEE, NIST, industry-specific standards
- **De Facto Standards**: Widely accepted practices without formal standardization
- **Compliance Requirements**: Regulations, certifications, audits
- **Quality Frameworks**: CMMI, Six Sigma, domain-specific quality frameworks
- **Acceptance Criteria**: How is "good enough" defined in this domain?

### Step 4: Catalog Common Pitfalls and Failure Modes
**Action**: Identify what commonly goes wrong in this domain
**Input**: Best practices + quality standards
**Output**: Pitfall catalog

Categories of pitfalls:
1. **Process Pitfalls**: Skipping steps, wrong order, insufficient review
2. **Quality Pitfalls**: Inadequate testing, missing edge cases, overconfidence
3. **Communication Pitfalls**: Ambiguous instructions, missing context, jargon confusion
4. **Scope Pitfalls**: Scope creep, over-engineering, under-engineering
5. **Domain-Specific Pitfalls**: Unique to the target domain

For each pitfall:
- **Description**: What goes wrong
- **Cause**: Why it happens
- **Impact**: How serious is it
- **Prevention**: How the steering policy should prevent it
- **Detection**: How to detect it happening

### Step 5: Document Domain-Specific Terminology
**Action**: Identify terms that need definitions in the target agent's glossary
**Input**: All research gathered
**Output**: Domain terminology list

For each term:
- **Term**: The domain-specific word or phrase
- **Definition**: Clear, concise definition
- **Context**: When/how it's used in the domain
- **Related Terms**: Similar or related concepts
- **Common Misuse**: How the term is often misunderstood

### Step 6: Identify Reference Implementations
**Action**: Find existing implementations or standards that can inform the policy design
**Input**: Best practices + quality standards
**Output**: Reference implementation catalog

Consider:
- Existing steering policies or workflow definitions in the domain
- Open-source tools with well-documented workflows
- Industry frameworks that define process steps
- Published methodologies or approaches
- AI-DLC patterns that map well to this domain

### Step 7: Generate Domain Research Questions (If Needed)
**Action**: Create questions to fill knowledge gaps
**Input**: All research with identified gaps
**Output**: Domain research question file

**IMPORTANT**: Follow overconfidence prevention (`common/overconfidence-prevention.md`).

Create `steering-docs/discovery/domain-research-questions.md` if:
- Domain is specialized and AI's knowledge may be incomplete
- User may have domain-specific insights not available through research
- Regulatory or compliance requirements need user confirmation
- Industry-specific practices vary by organization

### Step 8: Present Domain Research Summary
**Action**: Present findings to user for review and approval
**Input**: All research results
**Output**: Formatted summary for user review

---

## Output Artifacts

### Domain Research Summary
- **File**: `steering-docs/discovery/domain-research-summary.md`
- **Content**: Best practices, standards, pitfalls, terminology, references
- **Format**:

```markdown
# Domain Research Summary

## Domain Overview
- **Domain**: [domain name]
- **Research Depth**: [Minimal / Standard / Comprehensive]
- **Reasoning**: [Why this depth was chosen]

## Best Practices
### [Category 1]
- [Practice 1]: [Description]
- [Practice 2]: [Description]

### [Category 2]
- [Practice 1]: [Description]

## Quality Standards
| Standard | Relevance | Impact on Policy |
|----------|-----------|-----------------|
| [Standard 1] | [High/Medium/Low] | [How it affects the policy] |
| [Standard 2] | [High/Medium/Low] | [How it affects the policy] |

## Common Pitfalls
### [Pitfall 1]: [Title]
- **Description**: [What goes wrong]
- **Prevention**: [How the policy should prevent it]

### [Pitfall 2]: [Title]
- **Description**: [What goes wrong]
- **Prevention**: [How the policy should prevent it]

## Domain Terminology
| Term | Definition | Usage |
|------|-----------|-------|
| [Term 1] | [Definition] | [Context] |
| [Term 2] | [Definition] | [Context] |

## Reference Implementations
- [Reference 1]: [Description and how it informs our approach]
- [Reference 2]: [Description and how it informs our approach]

## Domain-Specific Considerations for Policy Design
1. [Consideration 1]
2. [Consideration 2]
3. [Consideration 3]

## Knowledge Gaps
- [Gap 1]: [What we don't know and why it matters]
- [Gap 2]: [What we don't know and why it matters]
```

---

## Completion Message

### REVIEW REQUIRED

**Domain Research is complete.** Here's a summary of findings:

- **Domain**: [domain name]
- **Research Depth**: [level]
- **Best Practices Found**: [N] across [N] categories
- **Quality Standards Identified**: [N]
- **Common Pitfalls Cataloged**: [N]
- **Domain Terms Documented**: [N]
- **Reference Implementations**: [N]

Key insights:
- [Insight 1]
- [Insight 2]
- [Insight 3]

### WHAT'S NEXT

Please choose one of the following:

**A) Request Changes** — I'll revise the domain research based on your feedback
**B) Continue to [Stakeholder Identification / Scope Definition]** — Proceed to the next stage

---

## Error Handling

### Unfamiliar Domain
- **Issue**: Domain is highly specialized with limited available information
- **Solution**: Switch to comprehensive depth, create detailed question file for user
- **Fallback**: Ask user to provide domain documentation, standards, or reference materials

### Conflicting Best Practices
- **Issue**: Different sources recommend different approaches
- **Solution**: Present alternatives with pros/cons, ask user for preference
- **Do Not Proceed**: Until approach is agreed upon

### Incomplete Research
- **Issue**: Cannot find sufficient domain-specific information
- **Solution**: Document gaps explicitly, ask user to fill in domain knowledge
- **Workaround**: Use closest analogous domain's practices as starting point

### Domain Scope Too Broad
- **Issue**: Domain research reveals multiple sub-domains with different requirements
- **Solution**: Ask user which sub-domain to focus on, or acknowledge breadth in scope definition
- **Recovery**: May need to revisit purpose analysis if scope changes significantly

## References
- `common/overconfidence-prevention.md` — When to ask vs assume
- `common/quality-standards.md` — Quality benchmark reference
- `common/terminology.md` — Base terminology to extend
