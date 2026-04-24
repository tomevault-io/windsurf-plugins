---
trigger: always_on
description: **BMAD (Business Modeling & Agent Development)** is a modular AI-agent platform that orchestrates specialized agents through structured workflows. The project implements **Merise Agile + TDD methodology** with 64 core mantras applied systematically across all agent creation and workflows.
---

# GitHub Copilot Instructions for BMAD Platform

## Project Overview

**BMAD (Business Modeling & Agent Development)** is a modular AI-agent platform that orchestrates specialized agents through structured workflows. The project implements **Merise Agile + TDD methodology** with 64 core mantras applied systematically across all agent creation and workflows.

**Key Modules:**
- **Core** (`_bmad/core/`): Platform foundation with party-mode, brainstorming, and base tasks
- **BMM** (`_bmad/bmm/`): Complete software development lifecycle (Analysis → Planning → Solutioning → Implementation)
- **BMB** (`_bmad/bmb/`): Meta-system for building agents, modules, and workflows
- **TEA** (`_bmad/tea/`): Test Architecture module (ATDD, automation, CI/CD, NFR assessment)
- **CIS** (`_bmad/cis/`): Creative Innovation & Strategy (design thinking, problem solving, storytelling)

## Architecture

### Agent System

**Agent Format:** Markdown files with YAML frontmatter + XML definitions
- Location: `_bmad/{module}/agents/{agent-name}.md`
- Structure: Frontmatter → Activation → Persona → Menu → Knowledge Base → Capabilities
- Activation is critical: Multi-step sequence that loads config, displays menu, waits for user input
- Menu handlers: `exec`, `workflow`, `tmpl`, `data`, `action`, `validate-workflow`

**Multi-Platform Support:**
- GitHub Copilot CLI (`.github/agents/*.md`)
- VSCode extensions
- Claude Code / Anthropic
- Codex platform (`.codex/prompts/*.md`)

**Agent Manifest:** `_bmad/_config/agent-manifest.csv` - Single source of truth for all installed agents

### Workflow System

**Workflow Format:** Markdown-based, multi-step, with YAML or MD steps
- Location: `_bmad/{module}/workflows/{workflow-name}/workflow.{md|yaml}`
- Steps: Separate files in `steps/` subdirectory for complex workflows
- Manifest: `_bmad/_config/workflow-manifest.csv` (name, description, module, path)
- Config resolution: `{project-root}`, `{output_folder}`, `{user_name}`, `{communication_language}`

**Workflow Types:**
- **Tri-modal:** Create / Validate / Edit (e.g., PRD, Architecture, Agents)
- **Sequential:** Multi-phase guided processes (e.g., Interview, Sprint Planning)
- **Standalone Tasks:** One-off utilities (e.g., Editorial Review, Shard Doc)

### Configuration System

**Module Configs:** Each module has `_bmad/{module}/config.yaml`
```yaml
user_name: Yan
communication_language: Francais|English
document_output_language: Francais|English
output_folder: "{project-root}/_bmad-output"
```

**Critical Variables:**
- `{project-root}`: Repository root
- `{planning_artifacts}`: `_bmad-output/planning-artifacts/`
- `{implementation_artifacts}`: `_bmad-output/implementation-artifacts/`
- `{project_knowledge}`: `docs/`

### Directory Structure

```
{project-root}/
├── _bmad/                          # Platform code
│   ├── _config/                    # Manifests (agents, workflows, tasks)
│   ├── _memory/                    # Agent persistent memory
│   ├── core/                       # Core module
│   │   ├── agents/
│   │   ├── workflows/
│   │   ├── tasks/
│   │   └── config.yaml
│   ├── bmm/                        # SDLC module
│   ├── bmb/                        # Builder module
│   ├── tea/                        # Test Architecture module
│   └── cis/                        # Innovation module
├── _bmad-output/                   # Generated artifacts
│   ├── planning-artifacts/
│   ├── implementation-artifacts/
│   └── bmb-creations/              # BYAN-created agents
├── .github/
│   └── agents/                     # GitHub Copilot agent definitions
└── .codex/
    └── prompts/                    # Codex/OpenCode agent definitions
```

## Code Conventions

### Agent Creation (BYAN/BMB Module)

**64 Mantras Applied:**
- **Mantra #37:** Ockham's Razor - Simplicity first, MVP approach
- **Mantra #39:** Every action has consequences - Evaluate before executing
- **Mantra IA-1:** Trust But Verify - Challenge all requirements
- **Mantra IA-16:** Challenge Before Confirm - Play devil's advocate
- **Mantra IA-23:** No Emoji Pollution - Zero emojis in code, commits, technical specs
- **Mantra IA-24:** Clean Code - Self-documenting, minimal comments

**Agent XML Structure:**
```xml
<agent id="agent-id" name="AgentName" title="Title" icon="🔧">
  <activation critical="MANDATORY">
    <step n="1">Load persona</step>
    <step n="2">Load config from {project-root}/_bmad/{module}/config.yaml</step>
    <step n="3">Store session variables</step>
    <step n="4-6">Display menu and wait for input</step>
  </activation>
  <persona>...</persona>
  <menu>...</menu>
  <capabilities>...</capabilities>
</agent>
```

### Workflow Creation

**Workflow Frontmatter:**
```yaml
---
name: workflow-name
description: Brief description
---
```

**Step Files:** Use numbered prefixes (`step-01-`, `step-02-`)

**Config Resolution:** Always resolve `{project-root}` and other variables from module config

### Merise Agile + TDD Principles

**Data Dictionary First:** Define all data entities before modeling (Mantra #33)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yan-Acadenice/BYAN](https://github.com/Yan-Acadenice/BYAN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
