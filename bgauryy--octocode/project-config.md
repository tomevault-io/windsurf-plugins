---
trigger: always_on
description: > Single source of AI agent guidance for the Octocode monorepo. Covers the root and every package — there is **no** per-package `AGENTS.md`.
---

# AGENTS.md — Octocode Monorepo

> Single source of AI agent guidance for the Octocode monorepo. Covers the root and every package — there is **no** per-package `AGENTS.md`.

## Contents

**Monorepo**
- [Documentation Links Rule](#documentation-links-rule)
- [Core Methodology](#core-methodology)
- [Repository Structure](#repository-structure)
- [Access Control](#access-control-monorepo-wide)
- [Quick Commands](#quick-commands)
- [Key References](#key-references)

**Packages**
- [`octocode-mcp`](#package-octocode-mcp) — MCP server (14 tools)
- [`octocode-cli`](#package-octocode-cli) — CLI installer + tool runner
- [`octocode-shared`](#package-octocode-shared) — Credentials, sessions, platform
- [`octocode-vscode`](#package-octocode-vscode) — VS Code extension
- [`octocode-security-utils`](#package-octocode-security-utils) — Security utilities

---

## Documentation Links Rule

All links in documentation files (`docs/`, package READMEs) **MUST** use absolute GitHub URLs — never relative paths.

**Base URL:** `https://github.com/bgauryy/octocode-mcp/blob/main/`

```
❌ WRONG: Config -> ./CONFIGURATION_REFERENCE.md
❌ WRONG: Auth -> ../docs/AUTHENTICATION_SETUP.md
✅ RIGHT: [Config](https://github.com/bgauryy/octocode-mcp/blob/main/docs/configuration/CONFIGURATION_REFERENCE.md)
✅ RIGHT: [Auth](https://github.com/bgauryy/octocode-mcp/blob/main/docs/configuration/providers/AUTHENTICATION_SETUP.md)
```

## Core Methodology

1. **Task Management**: Review → Plan (use `todo` tool) → Track progress
2. **Research**: Prefer `octocode-local` MCP tools. LSP first, then local search, then GitHub
3. **TDD**: Write failing test → Run (`yarn test`) → Fix → Verify coverage (90%)
4. **ReAct Loop**: Reason → Act → Observe → Loop
5. **Quality**: Clean Code, run `yarn lint` + `yarn test`, use `npx knip` for dead code
6. **Efficiency**: Use Linux commands (`mv`, `cp`, `sed`) for file operations

> **File Operations**: Use Linux commands for file changes and prefer batching changes.
> For command examples and workflows, see: [Linux & File Operations](https://github.com/bgauryy/octocode-mcp/blob/main/docs/dev/DEVELOPMENT_GUIDE.md#linux--file-operations)

## Repository Structure

```
octocode-mcp/
├── packages/
│   ├── octocode-mcp/             # MCP server: GitHub/GitLab/Bitbucket, local tools, LSP
│   ├── octocode-cli/             # CLI installer, tool runner, skills marketplace
│   ├── octocode-vscode/          # VS Code extension (OAuth, multi-editor MCP install)
│   ├── octocode-shared/          # Shared utilities (credentials, platform, session)
│   └── octocode-security-utils/  # Standalone security utilities (no AGENTS section)
├── skills/                       # AI agent skills (research, plan, roast, etc.)
├── docs/                         # ALL monorepo documentation (provider setup, references, workflows)
└── package.json                  # Workspace root (yarn workspaces)
```

## Access Control (monorepo-wide)

| Path | Access |
|------|--------|
| `packages/*/src/`, `packages/*/tests/` | ✅ Auto |
| `docs/` | ✅ Auto |
| `*.json`, `*.config.*` | ⚠️ Ask |
| `.env*`, `.octocode/`, `node_modules/`, `dist/`, `out/`, `coverage/` | ❌ Never |

## Quick Commands

Canonical command list lives in the [Development Guide](https://github.com/bgauryy/octocode-mcp/blob/main/docs/dev/DEVELOPMENT_GUIDE.md) (Commands & Workflow section).

## Key References

### Core
- **Docs Index**: [docs/README.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/README.md)
- **Configuration Docs**: [docs/configuration/](https://github.com/bgauryy/octocode-mcp/tree/main/docs/configuration) — install, auth providers, MCP clients, env/config, troubleshooting
- **Developer Docs**: [docs/dev/](https://github.com/bgauryy/octocode-mcp/tree/main/docs/dev) — tool/API references, workflows, architecture, contributing, skills
- **Specs**: [docs/specs/](https://github.com/bgauryy/octocode-mcp/tree/main/docs/specs) — design specs and RFCs
- **Development Guide**: [docs/dev/DEVELOPMENT_GUIDE.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/dev/DEVELOPMENT_GUIDE.md)
- **Configuration**: [docs/configuration/CONFIGURATION_REFERENCE.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/configuration/CONFIGURATION_REFERENCE.md)
- **Troubleshooting**: [docs/configuration/TROUBLESHOOTING.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/configuration/TROUBLESHOOTING.md)

### Octocode MCP
- **GitHub/GitLab/Bitbucket Tools**: [docs/dev/reference/GITHUB_GITLAB_TOOLS_REFERENCE.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/dev/reference/GITHUB_GITLAB_TOOLS_REFERENCE.md)
- **Local + LSP Tools**: [docs/dev/reference/LOCAL_TOOLS_REFERENCE.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/dev/reference/LOCAL_TOOLS_REFERENCE.md)
- **Clone & Local Workflow**: [docs/dev/workflows/CLONE_AND_LOCAL_TOOLS_WORKFLOW.md](https://github.com/bgauryy/octocode-mcp/blob/main/docs/dev/workflows/CLONE_AND_LOCAL_TOOLS_WORKFLOW.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgauryy/octocode](https://github.com/bgauryy/octocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
