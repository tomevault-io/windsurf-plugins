---
trigger: always_on
description: Guidance for Claude Code working in this repo. Specs in `openspec/specs/` are the authoritative contract; this file is the must-know-fast index.
---

# CLAUDE.md

Guidance for Claude Code working in this repo. Specs in `openspec/specs/` are the authoritative contract; this file is the must-know-fast index.

## Quick reference

| Need             | Command / path                                                               |
| ---------------- | ---------------------------------------------------------------------------- |
| Install          | `pnpm install` (run `corepack enable` first)                                 |
| Typecheck / lint | `pnpm run typecheck` · `pnpm run lint`                                       |
| Tests            | `pnpm test` (single file: `pnpm vitest run path/to/file.test.ts`)            |
| Dev stack        | `pnpm run dev:docker:up` (foreground, wipes+reseeds; `docs/docker.md`)       |
| OpenSpec         | `/opsx:propose` · `/opsx:explore` · `/opsx:apply` · `/opsx:archive`          |
| Spec provenance  | `pnpm run check:spec-provenance` (only `origin/main...HEAD`; CI is the gate) |
| Mutation check   | `node scripts/mutate.mjs --file … --spec … --mutation … --with …`            |

Conventional Commits required (commitlint). Pre-commit = lint-staged + `tsc --noEmit --incremental`. Pre-push = `pnpm test`. Never bypass git hooks with `--no-verify`.

PR titles and descriptions are always written in English (regardless of the language used in chat).

Operator surface = dashboard (`/dashboard/{tokens,projects,sessions,judgments,memories,consolidation,maintenance}`). No operator CLI; Docker image runs the server only.

## Claims need evidence

A finding is not a finding until it has been executed. Scale the evidence to the cost of being wrong: the higher the consequence, the less an argument from reading the code is worth. This generalises the standard `db-performance-auditor` already applies to queries — measure the alternative, don't assume it.

- **Never publish what you have not run.** Issue comments, security advisories, PR descriptions and spec text are outward-facing; a wrong claim there has to be retracted in public.
- **A subagent's finding is a hypothesis.** Re-verify before acting on it or repeating it — subagents report plausible readings as results.
- **Probe the boundary the real caller uses, and include a control that must pass.** A probe that skips a layer proves nothing about it (calling an MCP handler directly bypasses its zod schema, so "the tool accepts X" measured that way is not a fact about the tool). With only a failing case you cannot tell a real defect from a broken probe; the control is what tells you which you have.
- **Classify from behaviour, not from the symptom's shape.** "The observable result changes" makes something a behaviour change; whether it is a _defect_ depends on whether the current behaviour is defensible — same evidence bar as anything else.
- **A new guard is not covered until its test fails without it.** Weaken each condition separately and confirm the tests naming it go red — `scripts/mutate.mjs` does the backup/mutate/run/restore loop. Three tests in one session passed while proving nothing (one asserted over an empty result set; two short-circuited before the condition they named), and only mutation found them. A test green on both sides of the change is the default outcome, not the exception.
- **One instrument per series, named.** Never present an isolated statement's timing and an end-to-end operation's timing as one table: a 39× statement speedup was a 2.5× user-facing one, and the ratio came from silently switching instruments between rows. State which you measured, and quote the end-to-end figure when a user waits on it.

Where a claim is about spec text rather than runtime, the evidence is the verbatim quote with `file:line`, never a paraphrase.

## Architecture

Single Node process, single SQLite file. Server layers at `apps/server/src/{server,mcp,dashboard,services,db,consolidation,llm}/`. Shared plugin tree at `apps/plugin/` ships to FIVE clients (Claude Code, Codex CLI, Hermes Agent, opencode, Pi) — see [Plugin development](#plugin-development-discipline). Pi is the odd one out: it has no built-in MCP client (intentionally, per its own docs), so `.pi-plugin/` speaks Streamable HTTP MCP itself and discovers its tools with `tools/list` instead of enumerating them. Monorepo uses pnpm workspaces with `apps/*` (deliverables) and `packages/*` (shared libraries — empty for now, staged for future extractions).

### Load-bearing invariants (do NOT violate without an OpenSpec change)

- **Append-only memory.** Rows never `DELETE`d (narrow purge exceptions in `apps/server/src/services/{memory,agent-sessions}.ts` and `apps/server/src/scripts/seed-dev.ts`, allow-listed in `apps/server/src/test/invariants.test.ts`). `content` never `UPDATE`d. Lifecycle = `status` flips (`active` → `superseded` | `archived`) plus `replaces` links. Every consolidation op journaled in `consolidation_ops`, reversible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [susomejias/rembric](https://github.com/susomejias/rembric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
