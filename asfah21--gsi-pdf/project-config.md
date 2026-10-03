---
trigger: always_on
description: This file provides guidance to AI Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI Agents when working with code in this repository.

## Taskfile (Recommended)

This project uses [Task](https://taskfile.dev/) as a unified command runner. All build, dev, test, lint, and docker commands can be run from the repo root via `task <command>`. Run `task --list` to see all available commands.

Task `desc:` fields should describe **what** the task does, not **how** it does it. Keep them generic and stable: don't reference implementation details like aliases, internal helpers, mode flags, or which other task delegates to which. The description is for users picking a command from `task --list`, not a changelog of refactors.

### Quick Reference
- `task install` — install all dependencies
- `task dev` — start backend + frontend concurrently
- `task dev:all` — start backend + frontend + engine concurrently
- `task build` — build all components
- `task test` — run all tests (backend + frontend + engine)
- `task lint` — run all linters
- `task format` — auto-fix formatting across all components
- `task check` — full quality gate (lint + typecheck + test)
- `task clean` — clean all build artifacts
- `task docker:build` — build standard Docker image
- `task docker:up` — start Docker compose stack

## Comments

A comment must carry information the code cannot. If a reader could derive it from the code in front of them, delete it.

Comment the current state. Not what the code used to do, not what changed, not why it changed: git holds that. Where history explains the shape, state the reason instead, so "this used to reimplement the modal internals" becomes "thin wrapper over the shared Modal: duplicating its portal and focus trap is how dialogs drift apart". Future state goes in a TODO with an issue.

Write a comment when it does one of these four jobs:

- **Contract.** What a caller must know that the signature cannot say: preconditions, invariants, units, ownership and lifetime, thread-safety, error semantics, side effects. Document the contract of everything a caller outside the file can reach, and nothing else. Goes on the type/method/module as Javadoc, JSDoc, or a docstring.
- **Why.** The constraint the code satisfies, the bug it avoids, the alternative rejected and the reason.
- **Hazard.** "Must stay in sync with X", "order matters because Y", "do not remove, it prevents Z".
- **Map.** A short orientation at the top of a genuinely complex file: what it owns, and what it deliberately does not.

Never write:

- A comment that restates the next line. `// Handle drag start` above `handleDragStart` is noise.
- Section banners or position markers: `// --- Types ---`, `// Helpers`, `// =====`.
- Step narration in a function body (`// Step 1:`, `// Then we`). If the steps need labels they need names: extract functions. Numbering a genuinely numbered thing, like a wizard step, is fine.
- Commented-out code. Delete it.
- Doc tags that restate the signature. `@param blob - The blob` says nothing; omit the tag rather than pad it.
- Docs on self-explanatory members with no constraint to state.

Two tests before keeping a comment:

- **Delete it.** Is any information lost? If not, it stays deleted.
- **Could a name carry it instead?** A better identifier, an extracted function, or a named constant beats a comment. Prefer the code change.

A comment at the end of a line usually decodes that line, and that is worth keeping: `{0x25, 0x50} // "%PDF"`, `50L * 1024 * 1024 // 50 MB`. The rules that compare a comment against the code below it do not apply there, but a trailing TODO or a trailing bit of history is judged like any other.

A reference is supplementary, never load-bearing: the comment must survive deleting it. `// See #1234` is a dead end; `// saving first loses every annotation (#6865)` is not. Prefer a spec (`RFC 3161`) or CVE where one applies.

A TODO needs an issue, not an owner: `// TODO(#1234): re-enable the gate once account syncing lands`. If it is not worth an issue, it is not worth a TODO. A question is not a TODO.

A comment block over ~12 lines outside a file or type header usually means the code needs restructuring, or that the prose is product documentation and belongs in the docs repo.

`task comment-lint` checks the mechanical part of this on the lines you add, and runs inside `task pre-commit`. Reasoning, worked examples and the linter's own rules: @devGuide/CODE_COMMENTS.md

## Common Development Commands

### Build and Test
- **Build project**: `task build`
- **Run backend locally**: `task backend:dev`
- **Run all tests**: `task test` (or individually: `task backend:test`, `task frontend:test`, `task engine:test`)
- **Docker integration tests**: `./test.sh` (builds all Docker variants and runs comprehensive tests)
- **Code formatting**: `task format` (or `task backend:format` for Java only)
- **Full quality gate**: `task check` (runs lint + typecheck + test across all components)

After modifying any files in the project, you must run the relevant `task check` command that covers that area of the code. For example, when editing frontend files run `task frontend:check`; for Python engine files run `task engine:check`; for Java backend files run `task backend:check`.

### Docker Development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asfah21/GSI-PDF](https://github.com/asfah21/GSI-PDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
