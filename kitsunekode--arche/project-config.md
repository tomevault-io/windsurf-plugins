---
trigger: always_on
description: Read this file first, then the **nearest** local `AGENTS.md` for the workspace you are editing. Use [docs/README.md](docs/README.md) for user-facing commands and [`.docs/README.md`](.docs/README.md) for internal architecture/context. Load one matching [`.plans/active/`](.plans/active/) file only for approved in-flight work. Production deploy: [docs/deployment.md](docs/deployment.md) and [docs/deployment-env.md](docs/deployment-env.md).
---

# Agent guide (canonical)

Read this file first, then the **nearest** local `AGENTS.md` for the workspace you are editing. Use [docs/README.md](docs/README.md) for user-facing commands and [`.docs/README.md`](.docs/README.md) for internal architecture/context. Load one matching [`.plans/active/`](.plans/active/) file only for approved in-flight work. Production deploy: [docs/deployment.md](docs/deployment.md) and [docs/deployment-env.md](docs/deployment-env.md).

## Core priorities

**Performance first.** **Reliability first.** Keep behavior predictable under load and during failures (session restarts, reconnects, partial streams).

If a tradeoff is required, choose **correctness and robustness** over short-term convenience.

**Maintainability:** Before adding functionality, check whether shared logic belongs in a dedicated module. Duplicate logic across files is a code smell. Do not take shortcuts with one-off local hacks—refactor existing code when it improves the system.

## How to navigate

1. Nearest `AGENTS.md` (app or package you touch).
2. [docs/README.md](docs/README.md) — public/manual documentation.
3. [`.docs/README.md`](.docs/README.md) and one task-specific internal topic when implementing.
4. One matching [`.plans/active/`](.plans/active/) plan when executing approved work.

Run `bun run repo:doctor` before release or after large cleanup passes.

## Before push (required)

**Do not push until the minimum ladder passes** — including production **build**. On `main`, `prod`, or `develop`, run the full CI ladder instead.

GitHub Actions [`.github/workflows/ci.yml`](.github/workflows/ci.yml) runs the full ladder; failed pushes block the branch and waste review time.

### Minimum ladder (every push)

From the repo root:

```bash
bun run ci:min
```

That runs, in order: `format:check` → `turbo lint check-types` (all packages) → `bun test` → `turbo build` (all packages, including `apps/web` MDX generate + Next build).

On feature branches when the merge base is correct, the faster variant is fine:

```bash
bun run ci:min:affected
```

Run the same steps manually if you prefer:

| Step | Command                           | What it catches                                           |
| ---- | --------------------------------- | --------------------------------------------------------- |
| 1    | `bun run format:check`            | Oxfmt (use `bun run format` to fix)                       |
| 2    | `bunx turbo run lint check-types` | Oxlint + `tsc --noEmit` in every package with those tasks |
| 3    | `bun test`                        | All Bun tests (web, cli, packages, …)                     |
| 4    | `bunx turbo run build`            | Production builds — **required before push**              |

Pre-commit hooks run **lint-staged** (format + oxlint on staged files) and **gitleaks** on staged paths. They do **not** replace typecheck, test, or build.

### Full CI (protected branches and pre-review)

From the repo root:

```bash
bun run ci
```

Adds `pkg:check` and `repo:doctor:strict` on top of the minimum ladder.

On **push to `main` or `prod`**, CI does **not** use Turbo `--affected` — it typechecks and lints the **entire monorepo**. A change that only typechecks `apps/web` locally can still fail on `packages/registry`, `apps/cli`, etc. Always run full `bun run ci` before pushing to those branches.

For **feature branches / pull requests**, CI may use `--affected`, but full `bun run ci` is still safer before you ask for review:

```bash
bun run ci:affected
```

### Workspace-specific notes

- **`apps/web`:** After editing `content/docs/**` or `content/blog/**`, run `bun run --cwd apps/web mdx:generate` if `check-types` complains about `.source/`. Web tests: `bun test apps/web`.
- **`apps/cli`:** Covered by `pkg:check`; also `bun test apps/cli/tests` when touching CLI only.
- **`packages/*`:** Any new package dependency (e.g. web importing `@arche-template/registry`) must pass that package’s `check-types` in the full turbo graph.

### When to run doctor

- Before push to protected branches: included in `bun run ci`.
- After large cleanup, renames, or doc moves: `bun run repo:doctor` (warnings OK locally); use `repo:doctor:strict` before merge.

Prefer `AGENTS.md` over package `README.md` files for agent context. When behavior or commands change, update the nearest `AGENTS.md` and the owning public `docs/` or internal `.docs/` topic. Never treat `.plans/archive/` as current behavior.

## Stack map

| Workspace                 | Role                                                                           |
| ------------------------- | ------------------------------------------------------------------------------ |
| `apps/web`                | Next.js App Router; tRPC client + `trpcCaller` for RSC                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KitsuneKode/arche](https://github.com/KitsuneKode/arche) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
