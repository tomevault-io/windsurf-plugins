---
trigger: always_on
description: <!-- ============================================================
---

<!-- ============================================================
     STOP! READ THESE RULES BEFORE DOING ANYTHING
     ============================================================ -->

> **MANDATORY RULES**
>
> 1. **DISCUSS before implementing** - Answer questions first, code after
>    agreement
> 2. **Use `node` directly for TypeScript** - NOT tsx, ts-node, or tsc
> 3. **Check the platform** - Claude Code knows the OS (`Platform:` in system
>    prompt). Follow the platform-specific rules below.

---

# fork-sync Development Guide

Sync forked/vendored dependencies with upstream using Git 3-way merge with
AI-assisted conflict resolution.

## Quick Reference

```bash
yarn build        # Compile TypeScript to lib/
yarn test         # Run all tests (fast: skips slow integration tests)
yarn test:all     # Run all tests including slow integration tests
yarn lint         # Check linting (ESLint)
yarn format       # Check formatting (oxfmt)
yarn fix          # Fix everything: typecheck + format + lint:fix
```

**Note:** Claude/Copilot integration tests are slow (invoke external CLIs) and
are skipped by default. Use `yarn test:all` or set `RUN_INTEGRATION=1` to
include them.

## Important: Discuss Before Implementing

**When the user asks a question, ANSWER the question and DISCUSS solutions with
them.** Do not immediately jump to writing code or creating files. Have a
conversation first to understand what approach they prefer, then implement only
after getting agreement.

## Platform-Specific Rules

Claude Code reports the current platform in the system prompt as `Platform:`.
Check it and follow the appropriate rules below.

### Windows (`Platform: win32`)

**Claude Code's Bash tool runs bash even on Windows.** Bash does not understand
Windows paths or `.cmd` files natively. Wrap commands in `powershell -Command`:

```bash
# CORRECT:
powershell -Command "cd 'e:\GitHub\microsoft\rnx-kit\incubator\fork-sync'; yarn test"
powershell -Command "cd 'e:\GitHub\microsoft\rnx-kit\incubator\fork-sync'; yarn fix"

# WRONG - bash doesn't understand Windows paths:
cd e:\path; command                    # FAILS
cd /d "e:\path" && command             # FAILS
```

**Key pattern:** `powershell -Command "cd 'FULL_PATH'; command"`

#### The "nul" File Problem

On Windows, `nul` is a reserved device name. **NEVER redirect to `nul`** — it
creates an undeletable file:

```bash
# NEVER DO THIS on Windows:
command 2>nul         # Creates undeletable "nul" file!
command 2>/dev/null   # Same problem!
some_command > nul    # Same problem!

# CORRECT:
powershell -Command "command 2>`$null"
command 2>&1          # Or just merge streams
```

#### Forbidden Shell Commands on Windows

- **DO NOT** use `mkdir -p` (use `New-Item -ItemType Directory -Force`)
- **DO NOT** use `rm -rf` (use `Remove-Item -Recurse -Force`)
- **DO NOT** use `cat`, `grep`, `sed`, `awk` (use PowerShell equivalents or
  Claude Code's Read/Grep tools)
- **DO NOT** use Unix path separators in shell commands (use `\` or let Node.js
  handle paths)

### macOS / Linux (`Platform: darwin` or `Platform: linux`)

Standard bash commands work natively. No PowerShell wrapping needed:

```bash
cd /path/to/rnx-kit/incubator/fork-sync && yarn test
yarn fix
```

**Still avoid destructive commands** (`rm -rf`, `git clean -f`, etc.) without
user confirmation — these are dangerous on any platform.

## Running TypeScript Scripts

**Node.js 22.6+ runs TypeScript files directly** via built-in "type stripping" -
it removes type annotations and runs the resulting JavaScript. No compilation
step, no external tools needed.

```bash
# CORRECT - use node directly:
node src/sync.ts --help

# WRONG - do NOT use these:
# node --experimental-strip-types ...  # WRONG - flag not needed in Node 23.6+/22.18+
# npx tsx src/sync.ts                  # WRONG - external tool not needed
# npx ts-node src/sync.ts             # WRONG - external tool not needed
# tsc src/sync.ts                     # WRONG - compilation not needed
```

### Limitations

Type stripping only works with "erasable" TypeScript syntax. The following
features require `--experimental-transform-types`:

- `enum` declarations (use `const` objects or union types instead)
- Constructor parameter properties (`constructor(public x: number)`)
- `namespace` declarations
- `const enum`

## Project Structure

```
incubator/fork-sync/
  src/
    sync.ts              # Main sync CLI
    ai-merge.ts          # AI-powered merge conflict resolver CLI
    modules/             # Reusable modules
  test/                  # Unit tests (Node.js test runner)
  harness/               # Manual test harnesses
  ai-merge-prompt.md     # Merge conflict resolution prompt template
  lib/                   # Build output (gitignored)
```

### Main Scripts

- `src/sync.ts` - Dependency sync tool (published as `fork-sync` CLI)
- `src/ai-merge.ts` - Hunk-based merge conflict resolver (published as
  `ai-merge` CLI)
- `ai-merge-prompt.md` - Prompt template for AI conflict resolution

### Modules (`src/modules/`)

Reusable modules imported by the main scripts:

- `merge-hunks.ts` - Hunk parsing and coalescing logic
- `job-ui.ts` - Concurrent task scheduler with progress UI
- `claude.ts` - Claude CLI integration
- `copilot.ts` - GitHub Copilot CLI integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/rnx-kit](https://github.com/microsoft/rnx-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
