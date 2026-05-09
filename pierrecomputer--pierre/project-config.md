---
trigger: always_on
description: You must set `AGENT=1` at the start of any terminal session to enable
---

# PierreJS Monorepo

## Agent Environment

You must set `AGENT=1` at the start of any terminal session to enable
AI-friendly output from Bun's test runner:

```bash
export AGENT=1
```

## Tooling

- We exclusively use `bun` to run commands and install packages. Don't use `npm`
  or `pnpm` or `npx` or other variants unless there's a specific reason to break
  from the norm.
- Since we use `bun` we can natively run TypeScript without compilation. So even
  local scripts we run can be `.ts` files.
- We use Bun's `catalog` feature for dependencies in order to reduce differences
  in dependencies across monorepo packages.
  - **CRITICAL: NEVER add a version number directly to a package's**
    `package.json`. Always follow this two-step process:
    1. First, add the dependency with its exact version to the root
       `package.json` file inside `workspaces.catalog` (e.g.,
       `"new-package": "1.2.3"`)
    2. Then, in the individual package's `package.json`, reference it using
       `"catalog:"` (e.g., `"new-package": "catalog:"`)
  - **NEVER run `bun add <package>` inside a package directory** - this will add
    a version number directly which breaks our catalog pattern.
  - This rule is sometimes broken in packages that are published, in order to
    make sure that end-users aren't forced to our specific version. `apps/docs`
    would use the catalog version and `diffs` _may_ choose to use a range.
- npm "scripts" should work from inside the folder of the given package, but
  common scripts are often "mirrored" into the root `package.json`. In general
  the root scripts should not do something different than the package-level
  script, it's simply a shortcut to calling it from the root.

## Worktrees

We use `git worktree` to parallelize work. Each worktree lives at
`~/pierre/pierre-worktrees/<slug>/` and owns a **port offset** so dev servers,
E2E fixtures, and the Chrome remote-debug instance don't collide across
worktrees. The main clone keeps the historical default ports; only worktrees
shift.

The `bun run wt` command suite (defined in `scripts/wt.ts`) manages worktrees:

```bash
bun run wt new <slug>    # create a worktree, allocate offset, bun install
bun run wt rm <slug>     # kill its processes, remove the worktree
bun run wt clean         # kill zombie servers on all worktrees' ports
bun run wt ps            # show per-worktree port status (LISTEN / —)
bun run wt list          # summary of all worktrees (managed + external)
```

Dev scripts inside a worktree automatically pick up the offset through
`scripts/ws.ts`, which reads `<worktree>/.env.worktree` when invoked. Before
starting, they run `scripts/run-dev.sh` to kill any stale process bound to the
target port (zombies survive uncleanly-closed terminals, which is common under
AI agents).

**Cleanup contract for agents.** If you spin up dev servers, Playwright
fixtures, or Chrome debug instances inside a worktree, **you must run
`bun run wt clean` (or `bun run wt rm <slug>` if the worktree itself is being
torn down) before completing your turn.** This releases ports and kills spawned
processes so they don't accumulate across runs. Running `wt clean` with no
arguments cleans every managed worktree; prefer the targeted `wt clean <slug>`
when you know which worktree you were working in.

## Linting

We use `oxlint` at the root of the monorepo rather than per-package lint setups.

Run linting from the monorepo root:

```bash
bun run lint
bun run lint:fix
```

For CSS, we use `stylelint`:

```bash
bun run lint:css
bun run lint:css:fix
```

## Code formatting

We use `oxfmt` at the root of the monorepo.

Check formatting from the monorepo root:

```bash
bun run format:check
```

Apply formatting from the monorepo root:

```bash
bun run format
```

**Important:** Always run `bun run format` from the monorepo root after making
changes to ensure consistent formatting.

- Always preserve trailing newlines at the end of files.

## TypeScript

We use TypeScript everywhere possible and prefer fairly strict compiler
settings.

All projects should individually respond to `bun run tsc` for typechecking, but
many of those scripts are implemented with `tsgo` rather than plain `tsc`.

Shared compiler options live in the root `tsconfig.options.json` file.

The root `tsconfig.json` file is used to manage project references across the
monorepo.

We use project references between packages and apps.

- When adding a new package or app, update the root `tsconfig.json` references.
- When a package depends on another `workspace:` package, add the dependency to
  the consuming package's `references` block when needed for accurate and fast
  typechecking.

## Code readability

- When adding non-trivial helper functions, prefer a short comment directly
  above the function declaration that explains, in plain language, what the
  helper does and why it exists.
- Write these comments as if the reader is new to the codepath. Avoid vague
  shorthand like "snapshot" unless you immediately explain what data is being
  captured or derived.
- Prefer function-level comments over a lot of inline comments. Use inline
  comments only when a specific step inside the function is still non-obvious.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pierrecomputer/pierre](https://github.com/pierrecomputer/pierre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
