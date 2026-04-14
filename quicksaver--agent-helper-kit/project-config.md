---
trigger: always_on
description: This file provides context for AI agents working in this codebase.
---

# AGENTS.md

This file provides context for AI agents working in this codebase.

## Project Overview

**agent-helper-kit** is a VS Code extension (TypeScript) that helps developers move review context into chat and run agent-friendly shell workflows directly in the editor.

**Entry point:**

`src/extension.ts` → bundled to `dist/extension.js`.

## Documentation

- **INTEGRATION_SECURITY.md** describes the security model and integration points for LM tools, shell runtimes, and the approval system, mostly adapted from VS Code's core functionality.
- **README.md** at root, provides project features and description.

## Tech Stack

- **Language:** TypeScript
- **Build:** esbuild (`esbuild.mjs`)
- **Linting:** ESLint v10 flat config + TypeScript ESLint + stylistic + perfectionist
- **Formatting:** Prettier
- **Tests:** Vitest (`src/**/*.test.ts`)
- **Package manager:** Yarn 4 (`yarn.lock` present)

## Available Skills

Agent Skills are in `.github/skills/`, **activate** as many as fit the nature of your given task:

| Skill    | Use When                                                                                |
| -------- | --------------------------------------------------------------------------------------- |
| `vitest` | Writing unit and integration tests with Vitest, including test structure and assertions |

## Conventions and Constraints

- Use path alias `@/*` for imports from `src/*`.

## Type Safety

- Never use `any`
- Prefer type inference for function return signatures

## Learnings

> **Purpose**: Capture critical concise (1-3 lines) insights that are NOT obvious from code or included documentation, and will be helpful for future development and maintenance.
>
> **Focus**:
>
> - non-obvious gotchas, workarounds, and discoveries
> - repeated failure modes
> - decision-shaping constraints
> - design and structural patterns that encode project conventions, preserve consistency, protect APIs, or prevent recurring bad usage patterns
> - entries that explain why a pattern exists, not just what the code does
>
> **Discard**:
>
> - information already obvious from docs, symbol names, or standard library/framework behavior
> - issues normally surfaced by normal tooling, such as build errors, test failures, or lint warnings
> - examples as code blocks, link to real code examples in files instead
> - generic best practices unless their use here protects a real repository boundary or recurring failure mode
> - duplicate restatements of the same pattern in multiple areas
> - historical details unless they still affect current design or maintenance decisions
>
> **Entry Format**:
>
> ```
> - **[Area] Topic**: Brief insight (target <= 50 words). See \`SymbolName\` in <path/to/file>.
> ```
>
> Use `Area` as a stable keyword describing the part of the system affected.
> Examples: `build`, `compatibility`, `tests`, `runtime`, `cache`, `layout`, `style`, `dashboard`, `auth`, `blog`, `api`, `lib`, `react`, `analytics`.
>
> **Example**:
>
> ```
> - **[lib] Brevo v4 SDK**: Responses return parsed data directly (no `.body`/`.response`), and raw HTTP details require `.withRawResponse()`. See `submit` in lib/brevo/actions.ts.
> ```

- **[tools] LM Tool Registration**: Shell tools must stay declared in `package.json` and registered at runtime, and their user-facing metadata should keep coming from the manifest with a safe fallback when parsing fails. See `registerShellTools` in src/shellTools.ts.
- **[tools] Prompt Reference Names**: Any LM tool with `canBeReferencedInPrompt: true` also needs `toolReferenceName`, or VS Code rejects registration during activation. See `contributes.languageModelTools` in package.json.
- **[approval] Read-Only Rule Specificity**: Prefer subcommand-specific regex allow rules for shell approval instead of broad command-name allows; named rules over-approve write-capable variants like `git branch`. See `DEFAULT_APPROVAL_RULES` in src/shellToolSecurity.ts.
- **[extension] Feature Toggle Completeness**: Disabling a feature requires both runtime disposal and manifest `when`/`enablement` guards; unregistering alone leaves commands or chat tools visible after reload. See `activate` in src/extension.ts.
- **[runtime] Tool Boundary**: Resolve workspace-relative `cwd` and other editor-context defaults in the tool layer, then pass concrete values into `ShellRuntime`; this keeps runtime reusable and testable. See `resolveCommandCwd` in src/shellTools.ts.
- **[runtime] Single Execution Path**: Keep sync and async shell runs on the same background-command lifecycle so IDs, streaming reads, kill behavior, and the Shell Runs panel stay consistent. See `runInSyncShellTool` in src/shellTools.ts.
- **[runtime] Completion Drain**: Treat process `exit` as authoritative status, but finalize on `close` or a short drain timer so piped output arriving after exit is not lost. See `recordExit` in src/shellRuntime.ts.
- **[runtime] Public Shell IDs**: Expose only the suffix externally and generate random internal IDs; per-process counters collide across VS Code windows when persisted artifacts share a temp directory. See `createUniqueShellId` in src/shellRuntime.ts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Quicksaver) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
