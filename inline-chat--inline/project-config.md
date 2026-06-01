---
trigger: always_on
description: - Inline is a work chat app
---

## Orientation

- Inline is a work chat app
- Key paths: 
  - `server/` Backend (Bun/TS)
  - `apple/` macOS/iOS clients (SwiftUI/UIKit/AppKit)
  - `landing/` web app (React/TanStack, not started) 
  - `desktop/` Windows app (Electron, not started)
  - `proto/` protobufs
  - `cli/` client cli (Rust)
  - `packages/mcp/` MCP
  - `packages/openclaw/` OpenClaw plugin
  - `packages/bot-api/` bot API client (HTTP, TypeScript)
  - `packages/sdk/` realtime SDK (WS, TypeScript)
- Shared Swift packages: `apple/InlineKit`, `apple/InlineUI`, `apple/InlineProtocol`; app targets: `apple/InlineIOS`, `apple/InlineXIOS`, `apple/InlineMac`.
- Backend structure: `src/functions`, `src/realtime`, `src/db`.
- Use `bun` for JS/TS tooling (not npm/yarn); keep IDs as `Int64` (`Id`/`ID`) and timestamps in seconds unless API requires ms.
- Production: `https://api.inline.chat` and `https://inline.chat`.

## Critical Rules

- Never revert/discard/reset/clean work unless explicitly asked; ask before one-way deletion commands (`rm`, restore/reset/checkout) unless explicitly requested.
- When the worktree is dirty, continue without stopping for unrelated modified/untracked files. Only stop if unexpected changes appear in the specific file/hunk currently being edited.
- Never read, write or touch `.env` files.

## Multi-agent Safety

- Do not create/apply/drop stashes (including `--autostash`) unless explicitly requested.
- Do not switch branches, create/modify worktrees, or clone this repo for commit/push unless explicitly requested.
- On "push", `git pull --rebase` is allowed; if conflicts occur, stop and ask before resolving.
- Scope commits to your changes unless user asks for "commit all".
- When unrecognized files exist, continue and focus only on relevant files.
- When a file contains unrelated changes, only stage the hunks you changed manually. Do not blindly add files.

## Working Rules

- Use existing logging for production logs (`Log`, `server/src/utils/log.ts`).
- Avoid unsafe Swift/typescript patterns (`Any`/`any` where avoidable, force unwraps, `try!`, unsafe casts).
- Prefer minimizing dependencies and external tooling; do not add new dependencies unless the ongoing maintenance cost is clearly justified.
- Keep commits atomic and scoped; do not amend existing commits.
- Before committing, verify staged files (`git diff --cached --name-only`); prefer `scripts/committer "<msg>" <file...>`.
- Write commit message in lowercase, include scope like this: `macos: fix ...`; scopes include: macos|ios|server|mcp|sdk|cli|openclaw|apple|web|docs|website; for general changes either no prefix or generic ones: eg. chore
- If undoing your own changes in a file with other uncommitted edits, ask first.
- Regenerate protobufs when contracts change (`bun run generate:proto`); run focused `swift build` for touched Swift packages.
- Run focused tests/typechecks for affected areas; add/update tests for new features and regressions.
- When data has duplicated or denormalized representations, treat consistency as an invariant: identify the source of truth, update every write path through shared logic, and add regression tests for the invariant instead of fixing only one reader/UI.
- Web is WIP. Do not extend requested changes or investigations to `landing/` unless explicitly asked.
- New UI work must stay in new UI components; do not modify legacy sidebar/old UI.
- When asked to write a plan or save your investigation, make a file in `.context/` named `YYYY-MM-DD-title-kebab-case.md`.
- In final handoff/review/push, call out security risks, possible performance regressions, and state production readiness.
- Do just the right amount of engineering, not over engineer, and not under engineer. Simple and elegant solutions are often better than prematuraly complex solutions that go beyond the scope and spec.
- When adding colors, ensure the color supports light/dark theming and if it's not a one-off color, add it to the platforms theme class/module.
- Don't put implicit DB read/write calls in computed variables, keep them pure.
- Prefer simple neutral names for variables and types; avoid encoding setup/state history into names like `configuredDB` when `db` or `database` is enough.
- When commiting file(s) with multiple changes/fixes, add a concise bullet list of specific items in the commit description; if fixing regressions or tricky bugs in non-conventional ways, add a very short explaination in commit description. Keep it under ~300 characters.
- Do not remove comments explaining code or left by authors of code.
- Preserve intentional comments, including inline documentation, bug-fix references, explanations for commented-out code, reasons for tricky code, and future TODO/FIXME notes. Only remove comments when they are clearly stale or incorrect and the change requires it.

## Style Guide
- Use shorter function/variables names as much as possible, avoid long phrasal function names when we can keep it simply unless required. Don't use weird abbreviations like `idxes`; `msg`, `id`, `ctx` and alike are fine.
- Keep control flow simple by avoiding nested ifs as much as possible; prefer switch/matching and early return if applicable. 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inline-chat/inline](https://github.com/inline-chat/inline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
