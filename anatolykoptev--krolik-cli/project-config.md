---
trigger: always_on
description: > Internal guide for developing krolik-cli. For **usage** docs, see root `CLAUDE.md`.
---

# KROLIK CLI — Development Guide

> Internal guide for developing krolik-cli. For **usage** docs, see root `CLAUDE.md`.

**Version:** 0.7.0 | **Node:** >=20 | **Package Manager:** pnpm

## Architecture Principles

| Rule | Enforcement |
|------|-------------|
| SRP | One purpose per file |
| Max lines | 200 |
| Paths | Dynamic detection via `detectSrcPaths()`, never hardcoded |
| Dependencies | Explicit injection |
| Functions | Pure first |
| CLI flags | Extend existing, never add new |

## ⛔ FORBIDDEN: Regex for Code Analysis

**NEVER use regex to analyze or transform code.** Regex cannot reliably distinguish:
- JSX attributes vs object properties vs function arguments
- String literals in different contexts
- Nested structures and edge cases

| ❌ Forbidden | ✅ Required |
|-------------|-------------|
| Regex patterns for parsing | SWC for fast analysis (`lib/@swc`) |
| String matching for context | ts-morph AST for transformations (`lib/@ast`) |
| Line-by-line text replacement | AST-based node replacement |

**Why:** Regex leads to exponential edge cases (11+ patterns for i18n alone). AST gives 100% accuracy via `SyntaxKind`.

**Example — i18n string replacement:**
```typescript
// ❌ WRONG: Regex-based (breaks constantly)
if (/^\w+:\s*["']/.test(line)) { /* object property? maybe... */ }

// ✅ CORRECT: AST-based (always accurate)
if (parent.getKind() === SyntaxKind.PropertyAssignment) { /* definitely object property */ }
```

**See:** [ast-transformer.ts](src/lib/@i18n/ast-transformer.ts) for reference implementation.

<!-- krolik:start -->
<!-- version: 6.1.0 | auto-updated -->

## 🐰 Krolik

### Session Startup

**FIRST:** Call these tools at session start:

1. `krolik_status` `fast: true`
2. `krolik_mem_recent` `limit: 5`
3. `krolik_context` `feature: "..."` or `issue: "123"` (if working on specific feature)

### Context Cache

**FIRST:** Read `.krolik/CONTEXT.xml` — if missing, run `krolik_context -q`

```xml
<context mode="quick|deep|full" generated="ISO-timestamp">
```

| Mode | Sections | Use |
|------|----------|-----|
| `quick` | architecture, git, tree, schema, routes | Fast overview |
| `deep` | imports, types, env, contracts | Heavy analysis |
| `full` | all sections | Complete context |

**Refresh if:** file missing, stale (>1h), or wrong mode

### Tools

| When | Tool | Params |
|------|------|--------|
| **Need progress overview** | `krolik_progress` |  |
| **Session start** | `krolik_status` | `fast: true` |
| **Before feature/issue work** | `krolik_context` | `feature: "..."` or `issue: "123"` |
| **Need library API docs** | `krolik_docs` | `action: "search", query: "..."` |
| **Parse GitHub issue details** | `krolik_issue` | `number: "123"` |
| **Before writing utilities** | `krolik_modules` | `action: "search", query: "..."` |
| **API routes questions** | `krolik_routes` | — |
| **DB schema questions** | `krolik_schema` | — |
| **Code quality audit** | `krolik_audit` | — |
| **Quality issues found** | `krolik_fix` | `dryRun: true` first |
| **Find duplicates/structure** | `krolik_refactor` |  |
| **After code changes** | `krolik_review` | `staged: true` |
| **Get recent memories** | `krolik_mem_recent` | `limit: 5` |
| **Save decision/pattern/bugfix** | `krolik_mem_save` | `type: "decision", title: "..."` |
| **Search memories by query** | `krolik_mem_search` | `query: "authentication"` |
| **Multi-agent orchestration** | `krolik_agent` | `orchestrate: true, task: "..."` |

<!-- krolik:end -->

---

## Project Structure

```
krolik-cli/
├── src/
│   ├── bin/            # CLI entry point (cli.ts)
│   ├── cli/            # Command registration
│   ├── commands/       # Command implementations
│   │   ├── agent/      # Multi-agent orchestration
│   │   ├── audit/      # Code quality audit
│   │   ├── codegen/    # Code generation (templates, generators)
│   │   ├── context/    # AI-context generation (formatters, parsers, repomap)
│   │   ├── docs/       # Library documentation cache (Context7)
│   │   ├── fix/        # Auto-fix (fixers, analyzers, strategies, recommendations)
│   │   ├── init/       # Project initialization
│   │   ├── issue/      # GitHub issue parsing
│   │   ├── memory/     # Persistent memory (SQLite)
│   │   ├── refactor/   # Module refactoring (duplicates, migrations)
│   │   ├── review/     # Code review (git diff analysis)
│   │   ├── routes/     # tRPC routes analysis
│   │   ├── schema/     # Prisma schema analysis
│   │   ├── security/   # Security audit
│   │   ├── setup/      # Plugin installation (diagnostics, installers)
│   │   ├── status/     # Project diagnostics
│   │   └── sync/       # Sync operations
│   ├── config/         # Config loading (cosmiconfig)
│   ├── lib/            # Shared utilities (@-prefixed modules)
│   │   ├── @agents/    # Agent definitions & orchestration
│   │   ├── @ast/       # AST operations (ts-morph pool, SWC parser, fingerprinting)
│   │   ├── @cache/     # SQLite caching layer
│   │   ├── @claude/    # Claude Code integration (sections)
│   │   ├── @core/      # Foundation: fs, shell, logger, time, utils, constants
│   │   ├── @detectors/ # Issue detectors: lint, security, i18n, quality, patterns
│   │   ├── @discovery/ # File discovery: architecture, reusables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anatolykoptev/krolik-cli](https://github.com/anatolykoptev/krolik-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
