---
trigger: always_on
description: <!-- Generated: 2026-01-28 | Updated: 2026-01-31 -->
---

<!-- Generated: 2026-01-28 | Updated: 2026-01-31 -->

# oh-my-droid

Multi-agent orchestration system for Factory Droid CLI, providing intelligent delegation, parallel execution, and IDE-like capabilities through LSP/AST integration.

**Version:** 3.8.17
**Purpose:** Transform Factory Droid into a conductor of specialized AI agents
**Inspired by:** oh-my-zsh / oh-my-opencode

## Purpose

oh-my-droid enhances Factory Droid with:

- **32 specialized agents** across multiple domains with 3-tier model routing (Haiku/Sonnet/Opus)
- **37 skills** for workflow automation and specialized behaviors
- **31 hooks** for event-driven execution modes and enhancements
- **15 custom tools** including 12 LSP, 2 AST, and Python REPL
- **Execution modes**: autopilot, ultrawork, ralph, ultrapilot, swarm, pipeline, ecomode
- **MCP integration** with plugin-scoped tool discovery and skill loading

## Key Files

| File | Description |
|------|-------------|
| `package.json` | Project dependencies and npm scripts |
| `tsconfig.json` | TypeScript configuration |
| `CHANGELOG.md` | Version history and release notes |
| `docs/FACTORY.md` | End-user orchestration instructions (installed to user projects) |
| `src/index.ts` | Main entry point - exports `createDroidSession()` |
| `.mcp.json` | MCP server configuration for plugin discovery |
| `.factory-plugin/plugin.json` | Factory Droid plugin manifest |

## Subdirectories

| Directory | Purpose | Related AGENTS.md |
|-----------|---------|-------------------|
| `src/` | TypeScript source code - core library | `src/AGENTS.md` |
| `droids/` | Markdown prompt templates for 32 agents (see `droids/templates/` for guidelines) | - |
| `skills/` | 37 skill definitions for workflows | `skills/AGENTS.md` |
| `commands/` | 31 slash command definitions (mirrors skills) | - |
| `scripts/` | Build scripts, utilities, and automation | - |
| `docs/` | User documentation and guides | `docs/AGENTS.md` |
| `templates/` | Hook and rule templates (coding-style, testing, security, performance, git-workflow) | - |
| `benchmark/` | Performance testing framework | - |
| `bridge/` | Pre-bundled MCP server for plugin distribution | - |

## For AI Agents

### Working In This Directory

1. **Delegation-First Protocol**: You are a CONDUCTOR, not a performer. Delegate substantive work:

   | Work Type | Delegate To | Model |
   |-----------|-------------|-------|
   | Code changes | `executor` / `executor-low` / `executor-high` | sonnet/haiku/opus |
   | Analysis | `architect` / `architect-medium` / `architect-low` | opus/sonnet/haiku |
   | Search | `explore` / `explore-medium` / `explore-high` | haiku/sonnet/opus |
   | UI/UX | `designer` / `designer-low` / `designer-high` | sonnet/haiku/opus |
   | Docs | `writer` | haiku |
   | Security | `security-reviewer` / `security-reviewer-low` | opus/haiku |
   | Build errors | `build-fixer` / `build-fixer-low` | sonnet/haiku |
   | Testing | `qa-tester` / `qa-tester-high` | sonnet/opus |
   | Code review | `code-reviewer` / `code-reviewer-low` | opus/haiku |
   | TDD | `tdd-guide` / `tdd-guide-low` | sonnet/haiku |
   | Data analysis | `scientist` / `scientist-low` / `scientist-high` | sonnet/haiku/opus |

2. **LSP/AST Tools**: Use IDE-like tools for code intelligence:
   - `lsp_hover` - Type info and documentation at position
   - `lsp_goto_definition` - Jump to symbol definition
   - `lsp_find_references` - Find all usages across codebase
   - `lsp_document_symbols` - Get file outline
   - `lsp_workspace_symbols` - Search symbols across workspace
   - `lsp_diagnostics` - Get errors/warnings for single file
   - `lsp_diagnostics_directory` - Project-wide type checking (uses tsc or LSP)
   - `lsp_rename` - Preview refactoring across files
   - `lsp_code_actions` - Get available quick fixes
   - `ast_grep_search` - Structural code search with patterns
   - `ast_grep_replace` - AST-aware code transformation
   - `python_repl` - Execute Python code for data analysis

3. **Model Routing**: Match model tier to task complexity:
   - **Haiku** (LOW): Simple lookups, trivial fixes, fast searches
   - **Sonnet** (MEDIUM): Standard implementation, moderate reasoning
   - **Opus** (HIGH): Complex reasoning, architecture, debugging

### Modification Checklist

#### Cross-File Dependencies

| If you modify... | Also check/update... |
|------------------|---------------------|
| `droids/*.md` | `src/droids/definitions.ts`, `src/droids/index.ts`, `docs/REFERENCE.md` |
| `skills/*/SKILL.md` | `commands/*.md` (mirror), `scripts/build-skill-bridge.mjs` |
| `commands/*.md` | `skills/*/SKILL.md` (mirror) |
| `src/hooks/*` | `src/hooks/index.ts`, `src/hooks/bridge.ts`, related skill/command |
| Agent prompt | Tiered variants (`-low`, `-medium`, `-high`) |
| Tool definition | `src/tools/index.ts`, `src/mcp/omc-tools-server.ts`, `docs/REFERENCE.md` |
| `src/hud/*` | `commands/hud.md`, `skills/hud/SKILL.md` |
| `src/mcp/*` | `docs/REFERENCE.md` (MCP Tools section) |
| Agent tool assignments | `docs/FACTORY.md` (Agent Tool Matrix) |
| `templates/rules/*` | `src/hooks/rules-injector/` if pattern changes |
| New execution mode | `src/hooks/*/`, `skills/*/SKILL.md`, `commands/*.md` (all three) |

#### Documentation Updates (docs/)

| If you change... | Update this docs/ file |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MeroZemory/oh-my-droid](https://github.com/MeroZemory/oh-my-droid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
