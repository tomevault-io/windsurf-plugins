---
trigger: always_on
description: Lowdefy is a config-driven web framework built on Next.js. Apps are defined in YAML with **Blocks** (React components), **Operators** (logic functions like `_if`, `_get`), **Actions** (event handlers), and **Connections/Requests** (database/API integrations).
---

# Lowdefy Project Guide for Claude Code

Lowdefy is a config-driven web framework built on Next.js. Apps are defined in YAML with **Blocks** (React components), **Operators** (logic functions like `_if`, `_get`), **Actions** (event handlers), and **Connections/Requests** (database/API integrations).

## Documentation Navigation

| Location                  | Purpose                                                     | Audience                              |
| ------------------------- | ----------------------------------------------------------- | ------------------------------------- |
| **CLAUDE.md** (this file) | Coding standards, patterns, helpers                         | Claude Code when editing code         |
| **code-docs/**            | Internal architecture, design decisions, package deep-dives | Claude Code for understanding how/why |
| **packages/docs/**        | User-facing docs (Lowdefy app)                              | End users learning Lowdefy            |

**code-docs/** structure: `Overview.md`, `Philosophy.md`, `packages/`, `plugins/`, `architecture/`

**Claude Code Commands:** `/l-docs-init`, `/l-docs-package`, `/l-docs-plugin`, `/l-docs-architecture`, `/l-docs-update`, `/l-review-extract`, `/l-generate-tests`

## Repository Structure

```
packages/
├── api/              # Server-side API handling
├── build/            # App build pipeline
├── cli/              # Command-line interface
├── client/           # Client-side React components
├── engine/           # Core state and event engine
├── operators/        # Operator parsing system
├── plugins/
│   ├── actions/      # Action plugins (@lowdefy/actions-*)
│   ├── blocks/       # UI components (@lowdefy/blocks-*)
│   ├── connections/  # DB/API connectors (@lowdefy/connection-*)
│   └── operators/    # Operator plugins (@lowdefy/operators-*)
├── servers/
│   ├── server/       # Production server (@lowdefy/server)
│   └── server-dev/   # Development server (@lowdefy/server-dev)
└── utils/            # Shared utilities (@lowdefy/helpers, etc.)
```

## Server Architecture

| Package               | Purpose     | Entry Point       | Key Feature                             |
| --------------------- | ----------- | ----------------- | --------------------------------------- |
| `@lowdefy/server`     | Production  | `next start`      | Minimal, no watching                    |
| `@lowdefy/server-dev` | Development | `manager/run.mjs` | File watching, hot reload, auto-rebuild |

**server-dev manager** orchestrates: initial build → file watchers → server process → SSE-based hot reload. See `code-docs/architecture/` for details.

## Code Principles

### CRITICAL: Fix at the Source, Not the Symptom

**A guard clause is a code smell. Ask "why" before "how"**

1. **Ask: "Why is this happening here?"**
2. **Trace backwards** to find where the problem originated
3. **Fix at the source**

Example: If `connection.id.toLowerCase()` crashes because `id` is undefined, the fix is NOT `if (!connection.id) return`. The fix is ensuring invalid connections don't reach this code - e.g., schema validation should stop the build before processing invalid data.

**The urge to add a guard clause is a red flag that you're treating symptoms, not causes. The WHY needs to be understood before adding.**

### Build Does the Work, Runtime Stays Simple

Build validates, sets defaults, and always writes all artifacts (even as `{}`). Runtime should never need try/catch on imports, existence checks, or fallback defaults for build artifacts.

### Core Philosophy

Clarity over brevity. Explicit code over clever code.

### Simplification Balance

When refactoring or simplifying code:

- **Preserve functionality** - never change what code does, only how it does it
- **Eliminate redundancy** - remove unused code, duplicate logic, and unnecessary abstractions
- **Keep helpful abstractions** - don't remove abstractions that improve organization or testability
- **Single responsibility** - don't combine too many concerns into one function
- **Consider debuggability** - code should be easy to debug and extend; avoid dense one-liners

### Key Patterns

**One function per file** - Each file should export a single function, with the filename matching the function name:

```
buildConnections.js  → export default buildConnections
createCounter.js     → export default createCounter
validateBlock.js     → export default validateBlock
```

**Single object parameter with destructuring:**

```javascript
function buildConnections({ components, context }) {
  /* ... */
}
function createReadConfigFile({ directories }) {
  return async function readConfigFile(filename) {
    /* ... */
  };
}
```

**Safe iteration with nullish coalescing:**

```javascript
(components.pages ?? []).forEach((page) => {});
Object.keys(block.areas ?? {}).forEach((area) => {});
```

Prefer `??` over `||` - it only falls back on `null`/`undefined`, not falsy values like `0` or `''`.

**Build functions mutate and return `components`:**

```javascript
function buildX({ components, context }) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lowdefy/lowdefy](https://github.com/lowdefy/lowdefy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
