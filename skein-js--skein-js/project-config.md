---
trigger: always_on
description: Canonical guide for humans **and** AI coding agents working in this repo. If you only read
---

# AGENTS.md

Canonical guide for humans **and** AI coding agents working in this repo. If you only read
one file before contributing, read this one. (Claude Code: `CLAUDE.md` points here.)

## What skein-js is

The **open-source alternative to LangGraph Platform** (now LangSmith Deployment) **for TypeScript**:
a self-hosted [Agent Protocol](https://github.com/langchain-ai/agent-protocol) server for
[LangGraph.js](https://github.com/langchain-ai/langgraphjs), plus a CLI that is a **drop-in
replacement for the LangGraph CLI** (`skein dev` ⇄ `langgraph dev`, unchanged `langgraph.json`).
Think "[aegra](https://github.com/aegra/aegra) for TypeScript."

## Read first

- [docs/index.md](./docs/index.md) — overview & architecture
- [docs/reuse.md](./docs/reuse.md) — **what we reuse from LangGraph OSS vs. rebuild**
- [docs/code-practices.md](./docs/code-practices.md) — codified conventions
- [docs/testing.md](./docs/testing.md) — testing strategy
- [docs/roadmap.md](./docs/roadmap.md) — milestones

## Golden rules

1. **Reuse first.** Before writing anything, check whether a `@langchain/*` package already
   does it ([docs/reuse.md](./docs/reuse.md)), and whether a user could already solve it over the
   API we ship — if they nearly can, build the missing primitive, not the feature. The best code is
   the code we don't write, and published API is the code we can never unwrite: we rename by alias +
   `@deprecated` and never by removal, and a break majors every package at once.
   [`/audit-plan`](.agents/skills/audit-plan/SKILL.md) runs that check over a plan or proposal
   before any of it is built.
2. **Pragmatic functional style.** Pure core, dependencies injected; thin classes only for
   stateful resources (pools/clients). Immutable by default. Validate at boundaries with Zod;
   throw typed errors at the edges.
3. **Simple & consistent.** Small functions, named exports, kebab-case files, one public
   surface per package (`src/index.ts`). Match the surrounding style. Let the linter/formatter
   settle style — don't hand-argue it.
4. **Green before commit — never commit red.** A pre-commit hook enforces this mechanically:
   [`.githooks/pre-commit`](.githooks/pre-commit), wired by `pnpm install` (`prepare` sets
   `core.hooksPath`), runs `nx format:check` + `nx affected -t lint typecheck test` (with `CI=true`
   so Vitest runs once; `examples/*` excluded — they need live services / local `.env`) for whatever
   your staged changes touch. Build and the Docker-backed integration tests run in
   [CI](.github/workflows/ci.yml) — run them locally too for anything substantial
   (`nx run-many -t build test-integration`). Keep docs current with any behavior/architecture change —
   and when you touch a user-facing doc, regenerate the LLM bundle with `pnpm docs:llms` (rebuilds
   [`llms-full.txt`](llms-full.txt) from the curated list in `scripts/generate-llms-full.mjs`; the
   hand-written index is [`llms.txt`](llms.txt)).
   [`/commit`](.agents/skills/commit/SKILL.md) does the full local run + docs check for you. Only
   bypass the hook (`git commit --no-verify`) in a genuine pinch, and go green before you push. Same
   bar as [Definition of done](#definition-of-done).

## This is an Nx monorepo (pnpm) — use Nx

**Always drive tasks through Nx**, not ad-hoc scripts. Nx gives caching, the affected-graph,
and consistent targets across packages. Every root `pnpm` script is a thin wrapper over an
Nx target — there are no bare `eslint`/`prettier`/`vitest` invocations.

- `build` / `typecheck` / `test-integration` are defined per project in `project.json`.
- `lint` is inferred by the **`@nx/eslint`** plugin from the root `eslint.config.mjs`.
- `test` is inferred by the **`@nx/vite`** plugin from each project's `vitest.config.ts`.
- Formatting is **Nx's built-in Prettier** (`nx format:write` / `nx format:check`).

```bash
pnpm install                      # bootstrap the workspace

# whole workspace (each is `nx run-many -t <target>`)
pnpm build
pnpm typecheck
pnpm lint                         # nx run-many -t lint   (@nx/eslint)
pnpm format                       # nx format:write       (Prettier via Nx)
pnpm test                         # nx run-many -t test   (@nx/vite, unit)
pnpm test:integration             # nx run-many -t test-integration (Testcontainers; needs Docker)

# a single project
nx build core                     # == nx run core:build
nx test storage-postgres
nx test-integration storage-postgres

# only what your change affects (prefer this in CI + locally)
pnpm affected                     # nx affected -t lint test typecheck build
nx graph                          # visualize the project graph
```

### Tests

Vitest via a workspace config; Testcontainers for real Postgres/Redis. See
[docs/testing.md](./docs/testing.md).

```bash
pnpm test                         # fast unit + conformance (memory), no Docker
pnpm test:integration             # *.integration.test.ts — needs Docker
pnpm test:coverage
nx affected -t test               # affected projects only
```

### Adding a package

Match the existing shape exactly (keep it boring and consistent):
`packages/<dir>/{package.json, project.json, tsconfig.json, vitest.config.ts, README.md, src/index.ts}`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skein-js/skein-js](https://github.com/skein-js/skein-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
