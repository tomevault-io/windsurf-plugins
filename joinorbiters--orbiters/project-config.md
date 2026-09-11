---
trigger: always_on
description: Orientation for agents and for humans. Read this before touching anything at the
---

# AGENTS.md — working in the Orbiters monorepo

Orientation for agents and for humans. Read this before touching anything at the
root. Facts that are true of one project only live in that project's own
`projects/<name>/AGENTS.md`, which is the file you should also read when you work
there — both Claude Code and omp load the nearest one.

## What this repository is

One repository for every Orbiters project. PigroCRM is the first of them and, today,
the only one; it is a project in here, not the point of the place. Projects are
allowed to use different stacks, and are expected to share as much of their
dependency graph and their tooling as they honestly can.

```
projects/<name>/     everything one project owns: its apps, its packages, its docs,
                     its Dockerfiles, its compose file, its deploy scripts
shared/<name>/       code or assets used by more than one project
tooling/<name>/      configuration shared by every project
docs/                documentation about the monorepo itself, never about a project
```

`shared/brand` is the first of those and shows what belongs there: the palette, the
typeface and the brand mark, which the CRM and the website must agree on and neither
can own. A project small enough to be a single artifact may be one package at its own
root rather than growing an `apps/` directory with one entry in it, which is what
`projects/website` is. `tooling/` is still empty, and a directory is not created
before something real goes in it.

## The dependency rule, which is the whole reason these projects live together

**One `uv.lock` and one `pnpm-lock.yaml`, both at the root.** Two projects resolving
SQLAlchemy or TypeScript twice, at two versions, is the thing a monorepo exists to
make impossible.

- **Python**: `pyproject.toml` at the root is the uv workspace. Its `members` list
  names every package one by one rather than globbing, because `projects/*/apps/*`
  also matches `apps/web` — a Vite app with no `pyproject.toml` — and uv refuses to
  start on a member without one.
- **Node**: `pnpm-workspace.yaml` globs, because pnpm ignores a directory with no
  `package.json`. Its `catalog:` block is the single source of truth for
  build-and-test toolchain versions. A package writes `"typescript": "catalog:"`.
  **What a project ships to its users stays in that project's own `package.json`**;
  what it needs in order to be built, linted and tested belongs in the catalog.

One resolution means one version of a library for everybody. That is the point, and
it has a cost: a project that genuinely needs an incompatible pin has to leave the
workspace and carry its own lock. That is an exception with a reason written down,
never a default.

## Commands

Everything runs from the repository root.

```
uv sync --frozen                       # one virtualenv for every Python package
uv run ruff check projects/pigrocrm    # lint one project
uv run mypy                            # every Python source root (see [tool.mypy] files)
uv run pytest -q projects/pigrocrm/packages/core/tests   # narrow to what you touched

pnpm install --frozen-lockfile
pnpm --filter web lint
pnpm --filter web test
```

`pytest`'s `testpaths` and `mypy`'s `files` are at the root and name each project's
paths in full. Both resolve relative to the working directory rather than to the file
they are written in, which is why there is no per-project config file to `cd` into:
that only works when you happen to be in the right place, and silently checks
nothing when you are not. Narrow to one project by passing its paths as arguments.

`ruff` is the exception, because it really does resolve per file: the root
`ruff.toml` is the monorepo-wide style and each project extends it.

## Verification: three tiers, and where each check lives

1. **Local, before the PR exists** — `.github/preflight.json`. Everything expensive:
   the full Python suite, Playwright, the images. `preflight --list` prints what your
   diff selects before you trust it; `preflight --install-hook` runs it on push.
2. **`pull_request`** — one cheap gate per project, scoped by `dorny/paths-filter`.
3. **`push` to `main`** — the heavy tier (the corpus, the images), scoped by the
   same filters. It was unconditional until 2026-09-09, when the measurement said
   1,564 hosted minutes in eight days against a 2,000/month allowance on a private
   repository. The repository went public 2026-09-10 and hosted minutes are free now,
   but the scoping stays: a fifteen-minute trunk run on a docs commit is still a cost,
   just in waiting rather than in money (`docs/design/DECISIONS.md`, 2026-09-10). A
   push that cannot reach a project does not pay for that project's suite. When a push
   has no reachable base commit the filters are skipped and everything runs, and so
   does a release tag (`<project>-v<semver>`, which `ci.yml` also listens to): a
   production deploy is gated on the run of the tag itself, since the trunk's run for
   the same commit may have skipped every job of that project and still concluded
   green. A nightly `schedule` run on `main` (`17 3 * * *` UTC) earns the same full
   run for a different reason, buying back "the trunk proves the whole tree" without

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joinorbiters/orbiters](https://github.com/joinorbiters/orbiters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
