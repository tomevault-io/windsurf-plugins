---
trigger: always_on
description: A multi-agent development framework for Claude Code.
---

# The Agency

A multi-agent development framework for Claude Code.

## What is The Agency?

The Agency is a convention-over-configuration system for running multiple Claude Code agents that collaborate on a shared codebase. It provides:

- **Workstreams** - Organized areas of work (features, infrastructure, etc.)
- **Agents** - Specialized Claude Code instances with context and memory
- **Principals** - Human stakeholders who direct work via instructions
- **Collaboration** - Inter-agent communication and handoffs
- **Quality Gates** - Enforced standards via pre-commit hooks

## Core Concepts

### Agents
Each agent has:
- `agent.md` - Identity, purpose, and capabilities
- `KNOWLEDGE.md` - Accumulated wisdom and patterns
- `WORKLOG.md` - Sprint-based work tracking
- `ADHOC-WORKLOG.md` - Out-of-plan work tracking

### Workstreams
Workstreams organize related work:
- Shared `KNOWLEDGE.md` across agents in the workstream
- Sprint directories for planned work
- Multiple agents can work on the same workstream

### Principals
Human stakeholders who provide direction:
- Requests (`REQUEST-principal-XXXX`) - Directed tasks
- Artifacts - Deliverables produced for principals
- Preferences - How they like to work

### Collaboration
Agents communicate via:
- `./tools/collaborate` - Request help from another agent
- `./tools/news-post` / `./tools/news-read` - Broadcast updates
- `./tools/nit-add` - Flag issues for later

## Directory Structure

```
CLAUDE.md                    # This file - the constitution
claude/
  agents/                    # Agent definitions and context
    captain/                 # The captain - your guide (ships with The Agency)
    collaboration/           # Inter-agent messages
  workstreams/               # Workstream knowledge and sprints
    housekeeping/            # Default workstream
  principals/                # Human stakeholders
  docs/                      # Guides and reference
  logs/                      # Session and activity logs
  claude-desktop/            # Claude Desktop / MCP integration
tools/                       # CLI tools for The Agency
```

## Tools

**Session:**
- `./tools/myclaude WORKSTREAM AGENT` - Launch an agent
- `./tools/welcomeback` - Session restoration
- `./tools/session-backup` - Save session context

**Scaffolding:**
- `./tools/workstream-create` - Add a new workstream
- `./tools/agent-create` - Add a new agent
- `./tools/epic-create` - Plan major work
- `./tools/sprint-create` - Plan sprint work

**Collaboration:**
- `./tools/collaborate` - Request help
- `./tools/collaboration-respond` - Respond to requests
- `./tools/news-post` / `./tools/news-read` - Broadcasts

**Quality:**
- `./tools/commit-precheck` - Run quality gates
- `./tools/test-run` - Run tests
- `./tools/code-review` - Automated code review
- `./tools/review-spawn` - Generate review subagent prompts
- `./tools/install-hooks` - Install git pre-commit hooks

**Git:**
- `./tools/commit` - Create properly formatted commits
- `./tools/tag` - Tag work item stages (verifies tests pass)
- `./tools/sync` - Push with pre-commit checks

**GitHub:**
- `./tools/gh` - GitHub CLI wrapper (auto token injection + logging)
- `./tools/gh-pr` - PR operations (list, create, merge, etc.)
- `./tools/gh-release` - Release operations (list, create, view)
- `./tools/gh-api` - API operations (REST and GraphQL)

## Tool Output Standard

**All `./tools/*` must follow this output format to minimize context window usage.**

### stdout Format (What Claude Sees)

```
{tool-name} [run: {run-id}]
{essential-result-if-needed}
{status}
```

- **Line 1:** Tool name and run ID (for tracing to verbose logs)
- **Line 2:** Essential result only if needed (commit hash, file path, count)
- **Line 3:** Status indicator: `✓` (success) or `✗` (failure)

### Examples

```bash
# Success with essential result
commit [run: a1b2c3d4]
Committed: 9cbb97e
✓

# Success, no result needed
test-run [run: e5f6g7h8]
✓

# Failure
test-run [run: i9j0k1l2]
✗
```

### Verbose Output (Database)

Full output is captured in the database via `_log-helper`:
- stdout/stderr content
- Duration
- Exit code
- Arguments

**Investigate failures:**
```bash
./tools/agency-service log run {run-id}
```

### Why This Matters

| Location | Content | Token Impact |
|----------|---------|--------------|
| stdout (context) | 10-20 tokens | Minimal |
| Database | Full verbose output | Zero (not in context) |

Every token in stdout consumes context window. Verbose output is available when needed but doesn't waste tokens on successful runs.

## Terminal Integration

iTerm tab colors and status indicators update automatically via Claude Code hooks:
- **Blue ●** Available (ready for input)
- **Green ◐** Working (processing)
- **Red ▲** Attention (needs user input)

These are triggered automatically by hooks in `.claude/settings.json`. Do not call `./tools/tab-status` manually unless debugging.

**Reference:** See `claude/docs/TERMINAL-INTEGRATION.md` for setup and troubleshooting.

## Permissions

The Agency uses layered permissions:
- **`.claude/settings.json`** - Framework defaults (DO NOT EDIT - versioned with The Agency)
- **`.claude/settings.local.json`** - Your project permissions (gitignored - edit freely)

To add project-specific permissions (git, npm, domains):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-agency-ai/the-agency-starter](https://github.com/the-agency-ai/the-agency-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
