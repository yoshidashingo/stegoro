# Plugin Structure Generation — PACKAGING Phase

## Purpose
Generate Claude Code plugin structure from GENERATION/REFINEMENT-validated policy files. This stage transforms the quality-verified policy set into a distributable plugin format with agents, skills, commands, and rules.

## Prerequisites
- REFINEMENT Phase completed: 15 dimensions APPROVED (or CONDITIONAL APPROVAL with user consent)
- Scope Definition available (plugin structure design)
- All policy files generated and validated
- `common/implementation-knowhow.md` loaded for skill design patterns

## Execution Classification
**Type**: ALWAYS
This stage always executes as plugin packaging is the standard delivery format.

---

## Execution Steps

### Step 1: Load REFINEMENT Results
**Action**: Verify quality gate passage
**Input**: Quality Calibration scorecard
**Output**: Confirmed quality status

Verify:
- [ ] 15/15 dimensions PASS (APPROVED), or
- [ ] 13+ dimensions PASS with user-approved progression (CONDITIONAL APPROVAL)
- [ ] No unresolved repair loops

### Step 2: Load Scope Definition
**Action**: Load plugin structure design from Scope Definition
**Input**: Scope Definition document
**Output**: Plugin directory structure blueprint

Key information to extract:
- Agent name and directory structure
- Number of agents, skills, commands planned
- Plugin metadata (name, version, description)

### Step 3: Generate Plugin Configuration

#### 3a: plugin.json
Generate the plugin metadata file:

```json
{
  "name": "<agent-name>",
  "version": "0.1.0",
  "description": "<agent description from Purpose Analysis>",
  "agents": ["agents/*.md"],
  "skills": ["skills/*/SKILL.md"],
  "commands": ["commands/*.md"],
  "rules": ["rules/*.md"]
}
```

**Note**: Start with `0.1.0` for unreleased plugins; increment using semver (`0.x.y` → `1.0.0` for first stable release).

**Validation**:
- [ ] All required fields present (name, version, description, agents, skills, commands)
- [ ] Referenced paths will resolve after all files are generated
- [ ] Version follows semantic versioning (default: `0.1.0`)

#### 3b: marketplace.json
Generate marketplace publication metadata (per-plugin format — placed inside the plugin directory):

```json
{
  "displayName": "<User-friendly agent name>",
  "description": "<1-2 sentence description>",
  "category": "<domain category, e.g. 'methodology', 'productivity', 'engineering'>",
  "tags": ["steering-policy", "<domain-tag>"],
  "author": "<author>",
  "autoUpdate": true
}
```

**Note**: This is the **per-plugin metadata** file, not the marketplace registry format. The marketplace registry (`plugins: []` array wrapping) lives in the publisher's separate marketplace repo. This `marketplace.json` describes the plugin itself.

### Step 4: Generate Agent Definitions

Generate specialist agent definitions based on the target agent's workflow architecture.

**Standard Agent Set** (adapt to target agent):

| Agent | Role | Workflow Coverage |
|-------|------|-------------------|
| Orchestrator | Main workflow coordination, state management | All phases |
| Domain Specialist | Domain research, content expertise | DISCOVERY, GENERATION |
| Generator | Policy file generation | GENERATION |
| Quality Reviewer | Quality verification, scoring | REFINEMENT |

**For each agent file** (`agents/<agent-name>.md`):

```markdown
---
name: <agent-name>
description: <one-line description>
---

## Role
[Agent's primary responsibility]

## Capabilities
[List of what this agent can do]

## Workflow Integration
[Which phases/stages this agent handles]

## Tools and References
[MCP tools, rule-details files this agent uses]
```

### Step 5: Generate Skill Entry Points

Generate SKILL.md files (75-150 lines each) following the 10 skill design patterns from `implementation-knowhow.md`.

**For each skill** (`skills/<skill-name>/SKILL.md`):

**MANDATORY sections**:
1. **Activation Trigger** — When this skill is invoked (Pattern 1)
2. **Core Rules** — 4-5 memorable core rules (Pattern 2)
3. **Quality Gate** — Minimum quality criteria (Pattern 3)
4. **Workflow Summary** — Numbered phase/stage overview (Pattern 4)
5. **Anti-patterns** — What NOT to do (Pattern 5)
6. **References** — Paths to rule-details files

**GOOD Example**:
```markdown
---
name: steering-policy-creation
description: Create comprehensive steering policies for AI agents
---

## Activation Trigger
Use when: user requests steering policy creation, agent workflow design, or policy generation.
Keywords: "steering policy", "create policy", "agent workflow", "policy for"

## Core Rules
1. Discovery before Design — always analyze purpose and research domain first
2. Approval gates at every stage — never proceed without user confirmation
3. Domain specificity > 40% — generated content must be domain-tailored
4. 15-dimension quality — all policies calibrated against quality benchmark
5. Overconfidence prevention — ask when uncertain, never assume

## Workflow Summary
1. DISCOVERY: Purpose Analysis → Domain Research → [Stakeholder ID] → Scope Definition
2. DESIGN: Workflow Architecture → Common Rules → Phase Rules → Quality Mechanisms
3. GENERATION: Core Workflow → Common Rules → Phase Rules → Integration Validation
4. REFINEMENT: Completeness → Consistency → Quality Calibration (15 dimensions)
5. PACKAGING: Plugin Structure → Automated Validation → [Migration]

## Quality Gate
- All 15 quality dimensions PASS (or CONDITIONAL with user approval)
- Domain specificity rate >= 40% per Phase Rule file
- Integration validation 3-layer tests all PASS

## Anti-patterns
- Generating policies without domain research
- Skipping quality calibration
- Using generic templates without domain injection

## References
- `rule-details/core-workflow.md` — Complete workflow instructions
- `rule-details/common/` — Cross-phase rules (11 files)
- `rule-details/<phase>/` — Phase-specific rules
```

**BAD Example** (anti-pattern):
```markdown
---
name: policy-maker
description: Makes policies
---
# Just run the workflow
Follow core-workflow.md and do everything it says.
```
This fails because: no activation trigger, no core rules, no quality gate, delegates everything without guidance, too short to be useful.

### Step 6: Generate Commands

Generate command definition files (`commands/<command-name>.md`):

**Standard Commands**:

| Command | Purpose | Triggers |
|---------|---------|----------|
| `new-policy` | Start new policy creation | Fresh workflow start |
| `resume-policy` | Resume interrupted policy | Load state, continue |
| `validate-policy` | Validate existing policy | Quality check only |

**Command file format**:
```markdown
---
name: <prefix>:<command-name>
description: <one-line description>
---

## Execution
1. [Step with specific instruction]
2. [Step with specific instruction]

## Prerequisites
[What must be true before running]

## Output
[What this command produces]
```

### Step 7: Generate Rules File

Generate `rules/<agent-name>-standards.md` for quality enforcement:
- Reference to 15 quality dimensions
- Domain-specific quality rules
- File structure requirements

### Step 7b: Generate CLAUDE.md for Target Project

Generate a `CLAUDE.md` file for the target project repository. This file enforces Red Team review and provides project-level instructions for ongoing agent operations.

**Output location**: `steering-docs/<agent-name>/packaging/CLAUDE.md`

**Deployment**: Copy this file to the **root of the repository that will install and use the generated plugin** (i.e., the user's project, not the stegoro generator repo). The plugin itself lives at `<agent-name>/` within that project.

**Template**:

```markdown
# CLAUDE.md

このファイルは、Claude Code がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

<Purpose Analysis の目的分析サマリーを1〜2文で記述>

## エントリポイント

`<agent-name>/<agent-name>-rules/core-workflow.md` を最初に読み込んで指示に従うこと。

## 利用可能なツール

以下のMCPサーバーのみ使用可能です。他のMCPサーバーを使用しないでください。

| MCP Server | 用途 | 備考 |
|------------|------|------|
<Domain Research で特定したMCPサーバーを列挙>

## Red Teamレビュールール（必須）

Claude Codeがファイル作成・編集・設定変更等を出力した場合、**必ずCodexによるRed Teamレビューを実施すること**。

### レビュー手順

1. 出力完了後、`codex:codex-rescue` エージェントにRed Teamレビューを依頼する
2. レビュー観点:
   - **正確性**: 出力内容がプロジェクトの環境情報（README.md参照）と整合しているか
   - **安全性**: 機密情報（APIキー、認証情報等）が含まれていないか
   - **整合性**: 既存ファイルとの矛盾がないか
   - **完全性**: 必要な内容が欠落していないか
   - **ソース参照混入**: 他プロジェクトの固有情報が残っていないか
3. レビュー結果に問題がある場合は修正してから完了とする

### 適用範囲

- 新規ファイル作成時
- 既存ファイルの大幅な編集時
- エージェント設定変更時
- 外部サービスへの書き込み提案時（承認フローのPhase 4 Red Team Reviewとして実施）

## 言語

日本語で応答してください。
```

**Customization checklist**:
- [ ] プロジェクト概要を Purpose Analysis サマリーから記入
- [ ] MCPサーバー表を Domain Research 結果から記入（不要なら行ごと削除）
- [ ] エントリポイントパスを実際の `<agent-name>` に置換
- [ ] 適用範囲をエージェントのドメイン主要操作に合わせて追記・調整

**Validation**:
- [ ] `<agent-name>` プレースホルダーが全て実名に置換済み
- [ ] MCPサーバー表の備考欄に必要情報（スペースID・プロジェクトキー等）を記入済み
- [ ] Red Teamレビュー適用範囲がこのエージェントの主要操作を網羅している

### Step 8: Copy Rule Details

Copy improved rule-details files into the plugin structure:
- `<agent-name>-rule-details/common/` ← from generated common rules
- `<agent-name>-rule-details/<phase>/` ← from generated phase rules

### Step 9: Validate Plugin Structure

Before presenting for approval, verify:
- [ ] plugin.json has all required fields
- [ ] All referenced files in plugin.json exist
- [ ] All SKILL.md files are 75-150 lines
- [ ] All agent definitions have required sections
- [ ] All commands have execution steps
- [ ] Rule-details files are correctly copied
- [ ] `implementation-knowhow.md` 10 patterns checklist passes
- [ ] Working directory root (`<agent-name>/`) matches the source path used in Step 8 copy
- [ ] All file paths in SKILL.md, core-workflow.md, and agent definitions resolve to existing generated files

---

## Output Artifacts

### Generated Plugin Structure
- **Location**: `<agent-name>/`
- **Files**: plugin.json, marketplace.json, agents/*.md, skills/*/SKILL.md, commands/*.md, rules/*.md
- **Rule Details**: `<agent-name>-rule-details/` (copied from generation output)

---

## Completion Message

### REVIEW REQUIRED

**Plugin Structure Generation is complete.** Here's what was produced:

- **Plugin Directory**: `<agent-name>/`
- **Config Files**: plugin.json + marketplace.json
- **Agents**: [N] agent definitions
- **Skills**: [N] SKILL.md entry points (each 75-150 lines)
- **Commands**: [N] command definitions
- **Rules**: [N] rule files
- **Rule Details**: [N] files copied

### WHAT'S NEXT

**A) Request Changes** — I'll revise the plugin structure based on your feedback
**B) Continue to Automated Validation** — Proceed to 3-layer testing of the plugin

---

## Error Handling

### SKILL.md Line Count Exceeded (>150 lines)
- **Severity**: Medium
- **Solution**: Trim to core functionality. Move detailed content to rule-details files.
- **Recovery**: Review against Pattern 6 (content density limits)

### plugin.json Required Field Missing
- **Severity**: High
- **Solution**: Regenerate from template with all required fields
- **Recovery**: Validate against schema before writing

### Agent Definition / Core Workflow Inconsistency
- **Severity**: High
- **Solution**: Use core-workflow.md stage structure as source of truth
- **Recovery**: Regenerate agent definitions from workflow stages

### Skill Design Pattern Non-Compliance
- **Severity**: Medium
- **Solution**: Check against `implementation-knowhow.md` 10-pattern checklist
- **Recovery**: Add missing pattern elements (activation trigger, core rules, quality gate, etc.)

## References
- `common/implementation-knowhow.md` — Skill design patterns, plugin structure guide
- `common/output-structure-patterns.md` — Plugin file patterns
- `common/content-validation.md` — Plugin structure validation rules
