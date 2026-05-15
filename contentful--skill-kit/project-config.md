---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`@contentful/skill-kit` — TypeScript SDK for building agent skills with CLI-driven workflows. Companion to `@contentful/agents-kit`.

**Read `SPEC.md` before any substantive work.** It is the source of truth for the SDK's design, primitives, and protocol. The notes below are a reading guide, not a replacement.

## Tech stack

- **Runtime:** Node.js 24+ with tsx for dev
- **Package manager:** pnpm
- **Language:** TypeScript 5.9+ (strict mode, ESM)
- **Schema validation:** Zod 4
- **Test runner:** `node --test --import tsx/esm`, colocated `*.test.ts` files, `node:assert/strict`
- **Linting:** oxlint (correctness + suspicious rules; style rules off — Prettier owns formatting)
- **Formatting:** Prettier (`singleQuote: true`, `printWidth: 120`)
- **Build/distribution:** `--mode bun` (default) uses `bun build --compile` for standalone executables; `--mode node` uses esbuild for lightweight `.mjs` bundles

## Commands

- `pnpm install` — install dependencies
- `pnpm exec tsc --noEmit` — type check
- `node --test --import tsx/esm 'src/**/*.test.ts'` — run all SDK tests
- `node --test --import tsx/esm examples/get-to-know-you/src/skill.test.ts` — run workflow example tests
- `node --test --import tsx/esm examples/ts-patterns/src/skill.test.ts` — run reference example tests
- `node --test --import tsx/esm examples/contentful-help/src/skill.test.ts` — run composite example tests
- `pnpm run lint` — lint with oxlint
- `pnpm run lint:fix` — lint and auto-fix with oxlint
- `pnpm exec prettier --check .` — check formatting
- `pnpm exec prettier --write .` — fix formatting
- `node --import tsx/esm bin/skill-kit.js build <entry.ts> -o <outdir> --single` — build a skill executable (dev, current platform)
- `node --import tsx/esm bin/skill-kit.js build <entry.ts> -o <outdir> --mode node` — build a skill as a Node.js bundle

Example skills import `@contentful/skill-kit` which resolves via the `exports` field in the local `package.json`, not from a published npm package. After changing SDK source, run `pnpm run build` to compile the library before rebuilding examples.

## Conventions

- Follow agents-kit project conventions (no Nx)
- ESM only (`"type": "module"`)
- Tests colocated next to source files
- oxlint for correctness linting, Prettier for formatting (no ESLint)
- Published to GitHub Packages (`@contentful:registry=https://npm.pkg.github.com/`)

## Workflow

- **Always work on a branch** — never commit or push directly to main, not even single-line fixes. One branch per task, land via PR. Descriptive names: `feat/builder-api`, `fix/preamble-wiring`.
- **Conventional commits, committed frequently.** Commit the task doc first, then the code changes it produced. _Frequent_ means each logical stage gets its own commit as soon as it compiles on its own — not one giant `feat:` at the end containing a whole iteration. Target one commit per coherent slice: a refactor, a new pure module, a data-layer change, a test suite. When in doubt, commit. A reviewer should be able to read the branch top-to-bottom and follow the thinking; they should not have to diff 1200 lines in one shot.
- **Each commit must stand on its own.** Typecheck and tests pass at every commit — not just at the tip. If the stage you're committing depends on something you haven't written yet (e.g., a type export referenced by a module that doesn't exist), land the dependency first. This keeps `git bisect` useful and keeps `git revert` from unravelling the whole iteration.
- **Unrelated cleanups go in their own commit.** A `style:` / `chore:` commit for incidental Prettier or lint fixups on files you didn't otherwise touch. Don't smuggle them into a feature commit where they bloat the diff and muddle the history.
- **Task directories** for non-trivial work: `tasks/YYYY-MM-DD_hhmm_descriptive-kebab-case/TASK.md`. Get the timestamp from `date +%Y-%m-%d_%H%M` — do not guess. A task is a concrete, completable unit, not an epic.
- Every TASK.md has: **Scope** (what's in / explicitly out), **Context** (the _why_ — problem, constraint, user input that triggered it), **Plan** (approach + alternatives rejected + trade-offs), **Steps** (checkbox list), **Notes** (running log of decisions made _during_ implementation, written as you go).
- Task documents are the project's decision log. When someone later asks "why did we do X?", the answer should be findable in Context / Plan / Notes — not locked in a chat transcript.
- **The TASK.md is the only durable record.** Claude Code plan files (`~/.claude/plans/`) are ephemeral — they vanish when the session ends. Everything that matters for picking up an interrupted implementation must live in TASK.md: the agreed API/SDK design, type signatures, protocol changes, user feedback and design choices, rejected alternatives and why. Don't reference plan files from TASK.md — inline the substance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [contentful/skill-kit](https://github.com/contentful/skill-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
