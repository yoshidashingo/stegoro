<p align="center">
  <img src="docs/assets/logo.png" alt="stegoro logo" width="160">
</p>

<h1 align="center">
  stegoro
  <br><br>
</h1>

<p align="center">
  Automatically generate steering policies that drive purpose-built AI agents through structured, quality-assured workflows.
</p>

<p align="center">
  <a href="https://github.com/yoshidashingo/stegoro/blob/main/LICENSE"><img src="https://img.shields.io/github/license/yoshidashingo/stegoro" alt="license"></a>
  <img src="https://img.shields.io/badge/platform-Claude%20Code-blueviolet" alt="platform">
  <img src="https://img.shields.io/badge/quality-AI--DLC%2011%20dimensions-green" alt="quality">
</p>

<p align="center">
  <a href="README.md">English</a> &bull;
  <a href="README-ja.md">日本語</a>
</p>

## Why

Building a purpose-built AI agent is hard. Without a structured policy, agents produce inconsistent outputs, skip critical validation steps, and lack quality assurance. Writing these policies by hand is time-consuming and error-prone.

**stegoro** solves this by automating the entire policy creation process — from understanding the agent's purpose to generating a complete, quality-calibrated policy set.

## What

A meta-agent for [Claude Code](https://claude.ai/code) that generates steering policies for purpose-built AI agents. It produces a complete set of rule files that govern an agent's behavior through structured, multi-phase workflows.

### Features

- **4-Phase Workflow** — Discovery, Design, Generation, and Refinement phases ensure thorough policy creation
- **Adaptive Depth** — Automatically adjusts research and design depth based on domain complexity (Minimal / Standard / Comprehensive)
- **4 Agent Types** — Supports Process, Task, Analytical, and Hybrid agent patterns
- **AI-DLC Quality Assurance** — Validates output against 11 quality dimensions including adaptive workflow, mandatory checkpoints, audit trail, and overconfidence prevention
- **Interactive Dialogue** — Structured question-and-answer flow with approval gates at every critical decision point
- **Session Continuity** — State tracking enables pause and resume across sessions
- **Complete Audit Trail** — ISO 8601 timestamped logging of all decisions and user inputs

### Steering Policies as a User Harness

The generated policies function as a **user harness** — a structured control layer that wraps around the AI agent to ensure it executes complex workflows predictably and under human supervision.

Without a harness, an AI agent operates freely: it may skip validation steps, make autonomous decisions without approval, or deviate from the intended workflow. Steering policies solve this by defining explicit rules the agent must follow at every step.

**How the harness controls workflow:**

| Control Mechanism | How It Works |
|-------------------|--------------|
| **Approval Gates** | The agent cannot advance to the next phase or stage without explicit user confirmation |
| **Phased Execution** | Work is divided into discrete stages with defined inputs, outputs, and completion criteria |
| **Mandatory Checkpoints** | Critical decisions (scope, architecture, quality) always require user review before proceeding |
| **Overconfidence Prevention** | The agent is instructed to ask rather than assume when uncertain, preventing silent errors |
| **Audit Trail** | Every decision and user input is logged with ISO 8601 timestamps for accountability |

The `core-workflow.md` acts as the master orchestrator, routing the agent through each stage in order. The `rule-details/` files provide deep, domain-specific instructions that the agent reads on demand — keeping context focused and instructions precise. Together they ensure the agent behaves as a predictable, controllable system rather than a free-form AI.

### Packaging as an Agent Skill

The generated user harness can be packaged as a **Claude Code agent skill**, turning it into a distributable, installable plugin.

By adding a `.claude-plugin/` metadata directory alongside the generated policy files, the harness becomes a plugin that anyone can install and invoke via slash commands — no manual file setup required.

```bash
# Install a published agent skill
/plugin marketplace add <org>/<agent>
/plugin install <agent>@<agent>

# Invoke the agent
/<agent>:run
```

This is the deployment model used by the example agents in the [`samples/`](samples/) directory — each includes a `.claude-plugin/` directory that makes it publishable to the Claude Code marketplace.

### Generated Output

```
<agent-name>/                            # Claude Code plugin root (spec-compliant)
├── .claude-plugin/
│   └── plugin.json                     # Plugin metadata
├── agents/                             # Specialist agent definitions (optional)
├── commands/                           # Slash commands (optional)
└── skills/
    └── <skill-name>/                   # skills/<name>/SKILL.md (official pattern)
        ├── SKILL.md                    # Skill entry point (75-150 lines)
        ├── core-workflow.md            # Master orchestrator
        └── rule-details/               # Supporting files referenced from SKILL.md
            ├── common/                 # Cross-phase rules
            │   ├── welcome-message.md
            │   ├── process-overview.md
            │   ├── question-format-guide.md
            │   ├── content-validation.md
            │   ├── session-continuity.md
            │   ├── error-handling.md
            │   ├── terminology.md
            │   └── ...
            ├── <phase-1>/              # Phase-specific rules
            │   ├── <stage-1>.md
            │   └── ...
            └── <phase-N>/
                └── ...
```

**Note**: Quality standards live as supporting files inside `skills/<skill-name>/` per the official skills spec. The plugin's `marketplace.json` lives in the marketplace registry repo, not inside each plugin.

## How

### Prerequisites

- [Claude Code](https://claude.ai/code) installed and configured

### Installation

Add the marketplace and install the plugin (one-time setup):

```bash
/plugin marketplace add yoshidashingo/stegoro
/plugin install stegoro@stegoro
```

### Usage

1. Start the steering policy generator with a slash command:

```
/stegoro:generate a code review agent
```

More examples:

```
/stegoro:generate a proposal writing agent
/stegoro:generate a security audit agent
```

Or resume a previous session:

```
/stegoro:generate
```

2. The generator will guide you through the 4-phase workflow interactively:

| Phase | Purpose | Key Activities |
|-------|---------|----------------|
| **Discovery** | Understand what and why | Purpose analysis, domain research, stakeholder identification, scope definition |
| **Design** | Architect the policy | Workflow architecture, common rules, phase rules, quality mechanisms |
| **Generation** | Create the policy files | Core workflow, common rules, phase rules, integration validation |
| **Refinement** | Ensure quality | Completeness review, consistency review, quality calibration |

3. At each stage, review and approve before proceeding. The generator never advances without your explicit confirmation.

### Example Agents Built With This Tool

This repository includes example policies generated by the tool, located in the [`samples/`](samples/) directory:

| Agent | Description | Directory |
|-------|-------------|-----------|
| **Customer Support Agent** | Customer ticket triage, response, and quality review | [`samples/customer-support-agent/`](samples/customer-support-agent/) |
| **Proposal Writer** | RFP analysis and proposal creation | [`samples/proposal-writer/`](samples/proposal-writer/) |
| **Sales Pipeline Agent** | Sales pipeline management with cycle-based workflow | [`samples/sales-pipeline-agent/`](samples/sales-pipeline-agent/) |

## Documentation

Visit the [documentation site](https://stegoro.com/) for detailed usage guides.

## License

MIT
