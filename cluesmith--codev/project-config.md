---
trigger: always_on
description: > **Note**: This file is specific to Claude Code. An identical [AGENTS.md](AGENTS.md) file is also maintained following the [AGENTS.md standard](https://agents.md/) for cross-tool compatibility with Cursor, GitHub Copilot, and other AI coding assistants. Both files contain the same content and should be kept synchronized.
---

# Codev Project Instructions for AI Agents

> **Note**: This file is specific to Claude Code. An identical [AGENTS.md](AGENTS.md) file is also maintained following the [AGENTS.md standard](https://agents.md/) for cross-tool compatibility with Cursor, GitHub Copilot, and other AI coding assistants. Both files contain the same content and should be kept synchronized.

## Project Context

**THIS IS THE CODEV SOURCE REPOSITORY - WE ARE SELF-HOSTED**

This project IS Codev itself, and we use our own methodology for development. All new features and improvements to Codev should follow the SPIR protocol defined in `codev/protocols/spir/protocol.md`.

### Important: Understanding This Repository's Structure

This repository has a dual nature that's important to understand:

1. **`codev/`** - This is OUR instance of Codev
   - This is where WE (the Codev project) keep our specs, plans, reviews, and resources
   - When working on Codev features, you work in this directory
   - Example: `codev/specs/1-test-infrastructure.md` is a feature spec for Codev itself

2. **`codev-skeleton/`** - This is the template for OTHER projects
   - This is what gets copied to other projects when they install Codev
   - Contains the protocol definitions, templates, and agents
   - Does NOT contain specs/plans/reviews (those are created by users)
   - Think of it as "what Codev provides" vs "how Codev uses itself"

**When to modify each**:
- **Modify `codev/`**: When implementing features for Codev (specs, plans, reviews, our architecture docs)
- **Modify `codev-skeleton/`**: When updating protocols, templates, or agents that other projects will use

### Release Process

To release a new version, tell the AI: `Let's release v1.6.0`. The AI follows the **RELEASE protocol** (`codev/protocols/release/protocol.md`). Release candidate workflow and local testing procedures are documented there. For local testing shortcuts, see `codev/resources/testing-guide.md`.

### Local Build Testing

To test changes locally before publishing to npm:

```bash
# From the repository root:

# 1. Build (Tower stays up during this)
pnpm build

# 2. Pack, install globally, and restart Tower (one command)
pnpm -w run local-install
```

- `pnpm build` builds core first, then codev (including dashboard)
- `pnpm -w run local-install` runs `scripts/local-install.sh`, which:
  - Packs both `@cluesmith/codev-core` and `@cluesmith/codev` tarballs into their package directories
  - Globally installs both in one `npm install -g` (separate installs fail because `@cluesmith/codev-core` isn't on the public npm registry)
  - Restores the executable bit on `scripts/forge/**/*.sh` (pnpm pack strips it, causing "GitHub CLI unavailable" errors otherwise)
  - Restarts Tower so it picks up the new code
- Install runs while Tower is up — only the final restart causes downtime
- Do NOT stop Tower yourself before running the script — the script handles restart at the end
- Do NOT use `npm link` or `pnpm link` — it breaks global installs

### Testing

When making changes to UI code (tower, dashboard, terminal), you MUST test using Playwright before claiming the fix works. See `codev/resources/testing-guide.md` for Playwright patterns and Tower regression prevention.

## Quick Start

> **New to Codev?** See the [Cheatsheet](codev/resources/cheatsheet.md) for philosophies, concepts, and tool reference.

You are working in the Codev project itself, with multiple development protocols available:

**Available Protocols**:
- **SPIR**: Multi-phase development with consultation - `codev/protocols/spir/protocol.md`
- **ASPIR**: Autonomous SPIR (no human gates on spec/plan) - `codev/protocols/aspir/protocol.md`
- **AIR**: Autonomous Implement & Review for small features - `codev/protocols/air/protocol.md`
- **BUGFIX**: Bug fixes from GitHub issues - `codev/protocols/bugfix/protocol.md`
- **EXPERIMENT**: Disciplined experimentation - `codev/protocols/experiment/protocol.md`
- **MAINTAIN**: Codebase maintenance (code hygiene + documentation sync) - `codev/protocols/maintain/protocol.md`
- **RESEARCH**: Multi-agent research with 3-way investigation, synthesis, and critique - `codev/protocols/research/protocol.md`

Key locations:
- Protocol details: `codev/protocols/` (Choose appropriate protocol)
- **Project tracking**: GitHub Issues (source of truth for all projects)
- Specifications go in: `codev/specs/`
- Plans go in: `codev/plans/`
- Reviews go in: `codev/reviews/`

### Project Tracking

**GitHub Issues are the source of truth for project tracking.**

- Issues with the `spec` label have approved specifications
- Issues with the `plan` label have approved plans
- Active builders are tracked via `codev/projects/<id>/status.yaml` (managed by porch)
- The workspace overview Work view shows builders, PRs, and backlog derived from GitHub + filesystem state

**When to use which:**
- **Starting work**: Check GitHub Issues for priorities and backlog
- **During implementation**: Use `porch status <id>` for detailed phase status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cluesmith/codev](https://github.com/cluesmith/codev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
