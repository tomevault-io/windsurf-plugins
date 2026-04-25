---
trigger: always_on
description: forge-council is a pure-markdown multi-agent orchestration framework. It provides 13 specialist agents that run 3-round debates on topics, organized into four council types:
---

# Copilot Instructions for forge-council

## What This Project Does

forge-council is a pure-markdown multi-agent orchestration framework. It provides 13 specialist agents that run 3-round debates on topics, organized into four council types:
- **DeveloperCouncil**: Code review, architecture, debugging (6 dev specialists)
- **ProductCouncil**: Requirements, features, strategy (PM, Designer, Dev, Analyst)
- **DebateCouncil**: Cross-domain debate (SystemArchitect, UxDesigner, SoftwareDeveloper, WebResearcher)
- **KnowledgeCouncil**: Knowledge architecture and memory lifecycle decisions

No compiled code -- only markdown agent definitions, YAML configuration, and skills. Deployment uses Rust binaries from the forge-lib submodule.

## High-Level Architecture

### Core Components

**agents/** -- 13 specialist agents (each a markdown file with YAML frontmatter + structured instructions):
- SoftwareDeveloper, DatabaseEngineer, DevOpsEngineer, DocumentationWriter, QaTester, SecurityArchitect (dev track)
- SystemArchitect, UxDesigner, ProductManager, DataAnalyst (cross-domain)
- TheOpponent (strong model, devil's advocate)
- WebResearcher (web search + synthesis)
- ForensicAgent (strong model, PII and secret detection)

Each agent has:
- Frontmatter: `name` (PascalCase), `description`, `version`
- Deployment config (model, tools, scope) in `defaults.yaml`
- Body: Role, Expertise, Instructions, Output Format, Constraints

**skills/** -- 5 orchestration skills (each a directory with SKILL.md + SKILL.yaml):
- `/DebateCouncil` -- generic 3-round debate
- `/DeveloperCouncil` -- specialized code/architecture council
- `/ProductCouncil` -- specialized product/strategy council
- `/KnowledgeCouncil` -- knowledge architecture and memory lifecycle
- `/Demo` -- interactive showcase

**defaults.yaml** -- canonical agent roster, council compositions, and provider config. Single source of truth for deployment (model tiers, tools, scope).

**lib/** -- git submodule pointing to forge-lib. Provides Rust binaries:
- `bin/install-agents` -- multi-provider agent deployment
- `bin/install-skills` -- provider-aware skill installer
- `bin/validate-module` -- convention test suite

## Build, Test, Verify

```bash
make install          # deploy agents + skills for all providers (SCOPE=workspace|user|all)
make verify           # check agents deployed + skills present
make verify-skills    # verify skills across all runtimes
make test             # run module validation (validate-module)
make lint             # shellcheck all scripts
make clean            # remove previously installed agents
```

### Installation

**Standalone** (Claude Code plugin):
```bash
git clone --recurse-submodules https://github.com/N4M3Z/forge-council.git
cd forge-council
make install                           # workspace scope (default)
make install SCOPE=user                # user-level (~/.claude/agents/, etc.)
```

The Makefile automatically initializes the forge-lib submodule and builds Rust binaries on first run.

Standalone agent deployment without Make:
```bash
lib/bin/install-agents agents              # install all 13 agents
lib/bin/install-agents agents --dry-run    # preview without writing
lib/bin/install-agents agents --clean      # remove old agents, then install
```

**As forge-core module**:
```bash
Hooks/sync-agents.sh   # uses FORGE_LIB env var set by forge-core
```

### Verification

```bash
make verify            # all agents + skills for current SCOPE
make verify-agents     # 13 agents across claude/gemini/codex
make verify-skills     # skills across claude/gemini/codex/opencode
```

Deployed agents have `source:` frontmatter field pointing back to the source file.

Interactive check: `/Demo agents` in Claude Code to verify all specialists are recognized.

### Enable Parallel Council Execution (Optional)

Add to `~/.claude/settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

With this flag, councils spawn agents in parallel (TeamCreate + Task per agent). Without it, they run sequentially -- same verdict, slower.

## Key Conventions

### Agent Markdown Structure

Every agent file (`agents/*.md`) follows this structure:

1. **Frontmatter** (YAML between `---` delimiters):
   - `name`: PascalCase, must match filename (no .md)
   - `description`: Pattern: `"Role -- capabilities. USE WHEN triggers."` (USE WHEN clause for discoverability)
   - `version`: Semantic version (e.g., `0.3.0`)

   Deployment config (model, tools, scope) lives in `defaults.yaml`, NOT in agent frontmatter.

2. **Body** (in order):
   - Blockquote summary (one sentence, ends with "Shipped with forge-council.")
   - `## Role` -- what the agent does
   - `## Expertise` -- bullet list of domain areas
   - `## Personality` (optional, only for TheOpponent, WebResearcher, SecurityArchitect)
   - `## Instructions` -- detailed steps with `###` subsections
   - `## Output Format` -- markdown template in a fenced code block
   - `## Constraints` -- bullet list of boundaries

3. **Constraints section rules**:
   - Always end with: "When working as part of a team, communicate findings to the team lead via SendMessage when done"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [N4M3Z/forge-council](https://github.com/N4M3Z/forge-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
