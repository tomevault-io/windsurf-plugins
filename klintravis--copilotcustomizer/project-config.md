---
trigger: always_on
description: **Purpose**: This file provides global context to GitHub Copilot when working within the CopilotCustomizer toolkit itself. It establishes architectural patterns, file organization, workflows, and quality standards for all AI interactions.
---

# CopilotCustomizer Development Guide

**Purpose**: This file provides global context to GitHub Copilot when working within the CopilotCustomizer toolkit itself. It establishes architectural patterns, file organization, workflows, and quality standards for all AI interactions.

**Scope**: Applies to all `.github/` assets and toolkit maintenance. For external repository customizations, use the `/Bootstrap` workflow instead.

---

## Architecture Pattern

### Orchestrator + Subagents
`CopilotCustomizer.agent.md` routes ALL workflows through 6 specialized subagents via programmatic `agent` tool invocation (not manual handoffs).

**Workflow Chains**:
- **User Repositories**: Bootstrap → Planner → Generator → Editor/Verifier
- **Toolkit Self-Improvement**: Evolve → Editor → Verifier
- **Asset Creation**: Planner → Generator → Verifier
- **Targeted Edits**: Editor → Verifier

**Key Principles**:
- Fully programmatic orchestration — no manual handoffs
- State tracking via `todo` tool for multi-step workflows
- Quality gates between all phase transitions
- Context passing with explicit instructions to each subagent

---

## File Organization

```
.github/
├── agents/*.agent.md               # 7 agents (CopilotCustomizer + 6 subagents)
├── instructions/*.instructions.md  # 12 instruction files
├── prompts/*.prompt.md             # 10 slash commands
├── skills/[name]/SKILL.md          # 6 cross-platform skills (agentskills.io)
├── standards/[category]/           # 43+ standards across 8 categories
├── templates/                      # 4 structured output templates
├── hooks/subagent-tracking.json    # Lifecycle logging config (8 events)
├── scripts/log-orchestration.js    # Auto-invoked by hooks
└── logs/sessions/<timestamp>/      # Session-specific orchestration logs
```

### Asset Counts (Current)
- **Agents**: 7 (1 orchestrator + 6 subagents)
- **Instructions**: 12 (scoped via `applyTo` patterns)
- **Prompts**: 10 (user-facing slash commands)
- **Skills**: 6 (cross-platform, agentskills.io compatible)
- **Standards**: 43 (architecture, databases, devops, frameworks, languages, practices, security, testing)
- **Templates**: 4 (Analysis, ChangeLog, ImplementationPlan, OrchestrationPlan)

---

## Schema Compliance (VS Code v1.109)

### Agent Files (`.agent.md`)
**Required frontmatter**:
```yaml
description: "Brief description of agent role"
```

**Optional frontmatter**:
```yaml
tools: [editFile, createFile, runInTerminal]  # Tool access control
model: claude-sonnet-4                         # Model preference
agents: [Planner, Generator]                   # Subagent registry for handoffs
user-invokable: true                           # Allow direct user invocation
disable-model-invocation: false                # Enable via @agent syntax
```

**Deprecated**:
- ❌ `infer` field (removed in v1.109)
- Use `user-invokable: true` + `disable-model-invocation: false` instead

### Instruction Files (`.instructions.md`)
**Required frontmatter**:
```yaml
applyTo: ".github/**/*.agent.md"  # Glob pattern for file matching
```

**Optional frontmatter**:
```yaml
description: "Brief description of instruction purpose"
```

### Prompt Files (`.prompt.md`)
**Variable syntax**: `${VARIABLE_NAME}` (uppercase, underscores)

**Optional frontmatter**:
```yaml
agent: CopilotCustomizer           # Target agent for execution
tools: [readFile, searchFiles]     # Tool restrictions
model: claude-sonnet-4             # Model preference
```

**Variable types**:
- `${INPUT}` — Single-line text
- `${MULTILINE_INPUT}` — Multi-line text area
- `${FILE_PATH}` — File picker
- `${OPTION: "value1|value2|value3"}` — Dropdown selection

---

## Critical Workflows

### Asset Creation Workflows

| Command | Description | Execution Time | Output |
|---------|-------------|----------------|--------|
| `/NewAgent` | Create custom VS Code agent | ~45-60s | `.github/agents/[Name].agent.md` |
| `/NewInstructions` | Generate instruction file | ~30-45s | `.github/instructions/[Name].instructions.md` |
| `/NewPrompt` | Build slash command prompt | ~30-45s | `.github/prompts/[Name].prompt.md` |
| `/NewSkill` | Create cross-platform skill | ~45-60s | `.github/skills/[name]/SKILL.md` |
| `/NewAgentsFile` | Generate `AGENTS.md` | ~60-90s | `AGENTS.md` at repo root |
| `/NewMultiAgent` | Create orchestrated agent system | ~90-120s | Multiple agents + orchestration plan |

### Repository Workflows

| Command | Description | Execution Time | Output |
|---------|-------------|----------------|--------|
| `/Bootstrap REPOSITORY_PATH: "/path"` | Full autonomous asset generation | ~3-4 min | Complete `.github/` structure in target repo |
| `/Review` | Comprehensive asset audit | ~60-90s | Gap analysis + prioritized recommendations |
| `/QuickFix` | Fast targeted improvements | ~30-60s | Specific asset edits |

### Toolkit Workflows

| Command | Description | Execution Time | Output |
|---------|-------------|----------------|--------|
| `/Evolve FOCUS: "maintenance"` | Toolkit self-improvement | ~90-180s | Asset optimizations + harmonization |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/klintravis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
