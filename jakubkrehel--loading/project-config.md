---
trigger: always_on
description: This file is the single source of guidance for coding agents working in this repository. `CLAUDE.md` imports it and adds nothing but Claude Code specifics, so put repository facts here and do not maintain a second copy.
---

# AGENTS.md

This file is the single source of guidance for coding agents working in this repository. `CLAUDE.md` imports it and adds nothing but Claude Code specifics, so put repository facts here and do not maintain a second copy.

Note that `apps/web/AGENTS.md` and `examples/consumer/AGENTS.md` exist but are not yours to write — they hold only the managed `nextjs-agent-rules` block that `next dev` generates and re-adds. Leave them alone.

## Working agreement

Behavioural rules, not code conventions. They come from recurring corrections in this repository's sessions — treat them as project rules.

- **Change only what was named.** If a fix needs an adjacent file, helper, dependency or piece of copy, say so and stop. Removing the last usage of a dependency is not permission to uninstall it. Never delete or rewrite authored content — spinner prose, comments — as a side effect of another task.
- **A passing check is not a working feature.** `pnpm lint` proves the code compiles; it says nothing about how a spinner actually looks or moves. For any visual or runtime change, state what was verified and what was never seen running as two separate things. If it wasn't looked at, the word is "unverified" — a green check does not stand in for it, and never run a check just so there is something green to report.
- **Don't run builds or checks on routine changes.** A styling tweak, a spinner tweak, an MDX copy edit does not need `pnpm build` or `pnpm lint` afterwards. They cost more time than they save, and the dev server and editor surface the same errors sooner. Make the change and say what you changed. `pnpm build` is the worst offender — it builds every workspace package. Run a check only when the change is broad, touches config, types, the motion contract or the package exports, or when asked — and say in one line what it is for before starting it. Never narrate a step you are about to take instead of taking it.
- **End on the result.** No "want me to…", "say the word", "happy to…". If a decision is genuinely needed, ask it in the first line, not the last. A caveat earns its place only when it changes what to do next.
- **Edit prose, don't hedge it.** In `apps/web/src/content/spinners/*.mdx` and `README` copy, preserve the author's voice and level of certainty. No added qualifiers, no both-sides caveats, no new "never" absolutes.
- **Answer at the altitude of the question.** Reach for the platform primitive — a Tailwind media query over a custom hook, CSS over JS. The general solution is for after the specific one has been shown to fail.
- **Write commit and PR titles like a person.** `type(scope): short summary`, under ~60 characters — this repo scopes by package (`web`, `spinners`, `loading`), as in `fix(web): drop the theme background from the live snippet` and `refactor(spinners): unify spinner catalog and customization logic`. No trailing "for improved / for better / for consistency" clause. One change per title; if it needs an "and", it is two commits. Name the outcome, don't restate the diff, and avoid the filler verbs `enhance`, `streamline`, `standardize`, `optimize`.
- **PR descriptions are plain prose or nothing.** A few sentences on what changed and why; a `## heading` only when there is a real bug or decision to explain. Never a bulleted dump of the commit subjects. Most changes here need no body at all — leave it empty rather than padding it.

## Commands

All commands run from the repo root (`pnpm@11.8.0` workspace):

- `pnpm dev` — runs `tsup --watch` for the library and `next dev` for the site in parallel (`dev:lib` and `dev:web` run each alone)
- `pnpm build` — builds the library with tsup; `pnpm build:web` builds the site, which builds `loading-dev` first (`pnpm --filter loading-dev build && next build`)
- `pnpm lint` — Biome check (`biome check .`)
- `pnpm fix` — Biome check with autofix (`biome check --write .`)
- `pnpm format` — Biome format
- `pnpm typecheck` — checks library and test types, builds library declarations, then generates and checks site route types

`pnpm test` runs the Vitest suite in `tests/`. It renders every spinner in `SPINNERS` to static markup and checks the motion contract, so it needs no browser.

Domain vocabulary lives in `CONTEXT.md` — read it before naming anything.

## Architecture

pnpm workspace with the library at the root and one app beside it:

- **root** (`src/`, `tests/`) — the published npm package `loading-dev` ("Spinners. No more, no less."). React spinner components, ESM-only, built with tsup, React 19+ as a peer dependency. The root `package.json` is the package's manifest and also carries the workspace scripts and lint/test tooling; `files` limits the tarball to `dist`.
- **`apps/web`** — Next.js 16 (App Router, Turbopack, React Compiler enabled) showcase/docs site that consumes `loading-dev` via `workspace:*`.

Plus one directory that is **not** a workspace member:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakubkrehel/loading](https://github.com/jakubkrehel/loading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
