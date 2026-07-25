---
trigger: always_on
description: <!-- Source: .ruler/AGENTS.md -->
---



<!-- Source: .ruler/AGENTS.md -->

# Qwik v2 agents reference, instructions and rules

> [!IMPORTANT]
> **ALWAYS RUN `ruler apply` AT THE START OF A SESSION.** The agent skills are gitignored, not
> committed, so a fresh clone/worktree has none until you generate them (once per worktree):
>
> ```bash
> pnpm dlx @intellectronica/ruler@0.3.42 apply --no-gitignore --no-mcp
> ```

> Canonical source for repo-wide AI coding agent rules. For contributor setup, see
> [CONTRIBUTING.md](./CONTRIBUTING.md). For package-specific workflows, load the relevant
> `.ruler/skills/*/SKILL.md` file.

## Source Of Truth

- Shared AI guidance lives in `.ruler/`.
- Only the root `AGENTS.md` and `CLAUDE.md` are committed generated outputs; the `.claude/`/`.codex/`
  skill copies are gitignored and regenerated locally with `ruler apply` (see Setup below).
- Never hand-edit a generated output — edit `.ruler/` and regenerate with `ruler apply`. A CI check
  re-runs it and fails if the committed root files drift.
- To change assistant behavior, edit `.ruler/AGENTS.md`, `.ruler/README.md`, or `.ruler/skills/**`,
  then regenerate with Ruler when needed.

## Project Snapshot

Qwik is a resumable web framework. SSR serializes application and framework state into HTML, and
the client resumes without re-running component code. Qwik v2 is a rewrite with VNode-based
runtime work, rewritten reactive primitives, a new serialization mechanism, and package names under
`@qwik.dev/*`.

Key concepts: resumability, QRLs, `$`-suffixed optimizer boundaries, fine-grained signals, VNodes,
the cursor system, and the Rust optimizer.

## Monorepo Map

- Base branch and release branch for v2 PRs: `main`. V1 lives on the `v1` branch.

| Package | Path | Notes |
| --- | --- | --- |
| `@qwik.dev/core` | `packages/qwik` | Core runtime, SSR, optimizer-facing code |
| `@qwik.dev/router` | `packages/qwik-router` | Routing, middleware, adapters, SSG |
| `@qwik.dev/react` | `packages/qwik-react` | React integration |
| `@qwik.dev/dom` | `packages/qwik-dom` | Server-side DOM implementation |
| `@qwik.dev/qwik-vite` | `packages/qwik-vite` | `vite-plugin-qwik`, private, bundled into `@qwik.dev/core/optimizer` |
| `@qwik.dev/optimizer` | `packages/optimizer` | Rust optimizer, WASM, NAPI bindings |
| `eslint-plugin-qwik` | `packages/eslint-plugin-qwik` | ESLint rules |
| `create-qwik` | `packages/create-qwik` | Project scaffolding CLI |
| `qwik-docs` | `packages/docs` | Docs site, private package |
| `insights` | `packages/insights` | Analytics dashboard, private package |

Use v2 package names (`@qwik.dev/core`, `@qwik.dev/router`, etc.). Do not introduce v1
`@builder.io/qwik` or `@builder.io/qwik-city` imports except when working on explicit
compatibility override code.

## Setup And Iteration Loop

This is the canonical loop for nearly all framework work. Default to it; do not substitute broader
commands:

### getting started

```bash
pnpm i
```

```bash
# Materialize the agent skills into .claude/.codex (gitignored) — run once per fresh clone/worktree.
pnpm dlx @intellectronica/ruler@0.3.42 apply --no-gitignore --no-mcp
```

```bash
pnpm build.core # for a fresh start
pnpm build.full # for a fresh start and you are working on the docs (the docs run the optimizer)
```
### Iterating

Prefer focused commands and builds over repo-wide commands and builds.

```bash
# Fast dev build — required once before any tests can run, and again after framework
# source changes when the verification consumes build output (all e2e suites do)
pnpm build.core.dev

# Closest focused unit/spec test
pnpm vitest run packages/qwik/src/core/tests/use-task.spec.tsx

# Focused e2e test
pnpm playwright test e2e/qwik-e2e/tests/events.e2e.ts --browser=chromium --config e2e/qwik-e2e/playwright.config.ts
```

`build.core.dev` also re-emits fresh Qwik and Router `.d.ts` incrementally (via `tscDevDts` + re-export shims), so editing a public signature no longer leaves stale types — `build.watch` skips the type pass to stay instant.

For Qwik e2e tests, use `--browser=chromium` with `e2e/qwik-e2e/playwright.config.ts`.

Re-run `pnpm build.full` when you are touching the optimizer rust code.

### When making a PR

```bash
# for type-level verification when no focused test covers the change
pnpm tsc.check

# update the API
pnpm api.update

# Verify the build passes
pnpm build.core

# Verify unit tests pass
pnpm test.unit

# Verify the E2Es pass
pnpm test.e2e.chromium

# In case of html output change, update the ssg snapshot
pnpm test.e2e.router.ssg.update

# In case of a new feature, run the test.bench
pnpm test.bench

# In case of qwikloader changes
pnpm vitest packages/qwik/src/qwikloader.unit.ts -u
```

If any of those fail, fix and push your changes.

## Rules

Recent Qwik v2 work by core maintainers favors small, behavior-shaped changes with regression proof.
Follow that bias:

### Guidance Freshness

- If a skill or reference you used is stale, incomplete, or contradicted by current source, update
  the `.ruler` source guidance before finishing the task unless the user explicitly restricted the
  scope.
- Keep new durable lessons in the most specific skill or reference that future agents are likely to
  load. Do not add package-specific details to these always-on rules unless they affect most tasks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QwikDev/qwik](https://github.com/QwikDev/qwik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
