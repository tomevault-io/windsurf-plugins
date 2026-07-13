---
trigger: always_on
description: > This file gives AI agents the context they need to work on the OmAgents project.
---

# AGENTS.md

> This file gives AI agents the context they need to work on the OmAgents project.
> Read this before making any changes.

## What Is This Project?

OmAgents (`@omagents/omagents`) is an OpenCode plugin that bundles agent skills, MCP servers, parallel execution, and superpowers into a single npm package. Users install it by adding `@omagents/omagents` to their `opencode.json` plugin array.

## Architecture (Layered)

```
┌─────────────────────────────────────────────────┐
│  User Choice Layer (NOT bundled)                 │
│  OpenSpec · gstack · custom workflows · none     │
├─────────────────────────────────────────────────┤
│  Process Skills Layer (bundled: superpowers)     │
│  Brainstorming · TDD · Debugging · Plans ·       │
│  Code Review · Git Worktrees · Verification      │
├─────────────────────────────────────────────────┤
│  Infrastructure Layer (bundled: OmAgents)        │
│  MCP servers · Parallel execution ·              │
│  Deep research · Python tooling · venv           │
├─────────────────────────────────────────────────┤
│  OpenCode runtime                                │
└─────────────────────────────────────────────────┘
```

- **OmAgents** = infrastructure layer. Provides tools and capabilities.
- **Superpowers** = process skills layer. Provides development workflows.
- **User choice layer** is NOT bundled. OmAgents stays neutral on methodology.

## Project Structure

```
omagents/
├── .opencode/
│   ├── .gitignore              # Ignores node_modules, package.json, etc.
│   ├── package.json            # @opencode-ai/plugin SDK dependency
│   └── plugins/
│       ├── index.js            # Plugin entry point (merges superpowers + omagents hooks)
│       └── parallel.js         # Parallel execution engine (607 lines)
├── .github/
│   ├── ISSUE_TEMPLATE/         # bug_report.md, feature_request.md
│   └── workflows/
│       ├── ci.yml              # Syntax check on push/PR
│       └── publish.yml         # OIDC trusted publishing on tag push
├── skills/                     # Bundled OpenCode skills
│   ├── _shared/scripts/        # Shared scripts (loop_engine.py)
│   ├── deep-research/          # Multi-source research workflow
│   │   ├── SKILL.md
│   │   ├── agents/
│   │   ├── scripts/            # Python scripts (deep_research.py, plan.py, etc.)
│   │   └── templates/          # Jinja2 report templates (comparison, survey, technical)
│   ├── parallel-execution/     # Background task dispatch guide
│   │   └── SKILL.md
│   ├── agents-python-tools/    # Python venv management
│   │   └── SKILL.md
│   ├── markitdown-converter/   # Document to Markdown conversion
│   │   ├── SKILL.md
│   │   └── scripts/
│   └── playwright-web-scraping/# Web scraping with Playwright
│       ├── SKILL.md
│       └── scripts/
├── package.json                # superpowers as git dependency (pinned to commit)
├── package-lock.json
├── CHANGELOG.md
├── CONTRIBUTING.md
├── README.md
└── LICENSE
```

**IMPORTANT:** There is NO root-level `templates/` directory. Jinja2 templates live in `skills/deep-research/templates/`.

## Plugin Entry Point (`.opencode/plugins/index.js`)

The plugin does the following on load:

1. **Load superpowers** via `import("superpowers")` with graceful degradation
2. **Register skills** from `skills/` directory via `config.skills.paths`
3. **Register MCP servers** (see below)
4. **Merge hooks** from superpowers + parallel execution engine
5. **Provision Python venv** at `~/.venvs/omagents` on `session.created`, auto-installs `jinja2`
6. **Inject PATH** via `shell.env` hook: venv bin + skill script dirs + existing PATH

Key variables:
- `OMAGENTS_DIR` = project root (parent of `.opencode/`)
- `SKILLS_DIR` = `OMAGENTS_DIR/skills`
- `AGENT_VENV` = `~/.venvs/omagents`
- `AGENT_PYTHON` = `~/.venvs/omagents/bin/python`
- `SKILL_SCRIPT_DIRS` = script directories from deep-research, markitdown-converter, playwright-web-scraping

## Parallel Execution Engine (`.opencode/plugins/parallel.js`)

The parallel execution engine (607 lines):

- Intercepts `task` tool calls with `background: true`
- Maintains in-memory Job Board (`Map<taskID, JobRecord>`)
- Injects Job Board status into LLM context via `experimental.chat.messages.transform`
- Injects parallel execution system prompt via `experimental.chat.system.transform`
- Auto-enables background subagents by writing `OPENCODE_EXPERIMENTAL_BACKGROUND_SUBAGENTS=true` to shell config
- Provides custom tools: `parallel_status`, `cancel_task`
- Registers `/ps` command
- Writes TUI state to `~/.local/share/opencode/storage/omagents/tui-state.json`

**Known limitations:**
- Job Board is in-memory only (lost on restart)
- Job Board injects ALL sessions' jobs into every session (cross-session leak)

## MCP Servers

Registered automatically via `config` hook. User config takes precedence (won't override existing).

| MCP | Type | Config |
|-----|------|--------|
| `agentmemory` | local | `npx -y @agentmemory/mcp` |
| `codegraph` | local | `npx -y @colbymchenry/codegraph serve --mcp` |
| `context7` | remote | `https://mcp.context7.com/mcp` |
| `websearch` | remote | `https://mcp.exa.ai/mcp` |
| `github` | remote | `https://api.githubcopilot.com/mcp/` (requires `GITHUB_TOKEN`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omagents/omagents](https://github.com/omagents/omagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
