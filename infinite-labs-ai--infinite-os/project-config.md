---
trigger: always_on
description: Operating guide for anyone (human or AI agent) changing code in this repo. Pairs with
---

# AGENTS.md — working agreement for Infinite OS

Operating guide for anyone (human or AI agent) changing code in this repo. Pairs with
[CONTRIBUTING.md](CONTRIBUTING.md) (human-oriented) and [SECURITY.md](SECURITY.md).
If you read one thing: **work on a branch, open a PR, let CI gate it, get a review,
squash-merge.** Never push to `main`.

## What this repo is

**Infinite OS** — a self-hosted, local-first growth-analytics runtime. It connects a
user's data sources (GA4, PostHog, Stripe, Meta, Shopify, X) into a local Postgres
database, keeps them synced, and answers plain-English questions from a governed
metric layer. Everything runs on the user's machine; their data never leaves it.

Pure-TypeScript **pnpm-workspace monorepo**, Node ≥ 20:

| Path | What |
|---|---|
| `apps/cli` | the `infinite` command — operator shell + chat |
| `apps/app` | local HTTP API daemon the CLI talks to |
| `apps/worker` | scheduler + background sync jobs |
| `packages/*` | the engine (`db`, `core`, `config`, `connectors`, `runtime`, `metadata`, `analytical-engine`, `setup`, `llm-controller`, `instrument`) |
| `ui-tui/` | terminal UI (Ink) renderer |

> **`packages/instrument` is the published npm package [`infinite-tag`](https://www.npmjs.com/package/infinite-tag)** (NOT private). It's the founder-run installer that adds the GA4 / PostHog / X tracking tags into the *user's own website repo* (via `npx infinite-tag install` — public keys only, idempotent, reversible). `infinite setup` prints a pre-filled `npx infinite-tag install …` after analytics connect; GA4 also auto-installs it in-process (`packages/setup/src/provisioner.ts` → `import("infinite-tag")`). Don't reinvent tag-install logic — it lives here and is published via `.github/workflows/publish.yml`.

## The shipping workflow

`main` is protected: **no direct pushes, squash-merge only, CI (`ci`) must be green.**
Every change — including a maintainer's own — goes through a PR.

```bash
# 1. sync
git switch main && git pull --ff-only

# 2. branch (or a worktree for parallel work)
git switch -c <type>/<slug>            # type ∈ feat | fix | docs | chore | refactor
#   git worktree add ../io-<slug> -b <type>/<slug>

# 3. build
pnpm install
pnpm -r --if-present build             # or just run ./infinite once (it builds on first run)

# 4. verify — ALL must pass before opening the PR
pnpm typecheck                                 # tsc -b
pnpm test                                      # vitest suite (CI gates on a curated subset — see Tests)
PUBLIC_SURFACE=1 scripts/ci/repo-tripwire.sh   # no secrets / internal files tracked

# 5. PR
gh pr create --base main               # conventional-commit title; say what + why + tests run

# 6. review  → a separate pass reads the diff (CI green is necessary, not sufficient)
# 7. merge   → squash-merge, then delete the branch
```

CI also runs **gitleaks** over tracked content. External (fork) PRs run CI with
restricted permissions, and a maintainer approves the first run for new contributors —
fork PRs never receive repo secrets, and the tripwire degrades gracefully when a
secret-gated check can't run.

### Conventions

- **Conventional commits:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.
- One logical change per PR; keep PRs small and reviewable.
- New behavior ⇒ tests. Changed behavior ⇒ updated docs.
- **Author and review are separate passes.** Don't self-approve a change in the same
  breath you wrote it — have a fresh pass (a reviewer or a second read) check it.

## Versioning & releases

**Distribution model — git checkout, not npm.** `install.sh` clones this repo to
`~/.infinite/app` and drops the `infinite` launcher on the PATH. So "the latest
version" of the runtime is **the tip of your branch on `origin`** — there is *no npm
package to poll* for the CLI/runtime. (The only published npm package in this repo is
`infinite-tag` — see the name-collision warning below — and it is unrelated to runtime
versioning.)

- **The version number lives in the root `package.json`** (`version`), single source of
  truth — `infinite version` reads it. It follows **SemVer** (`MAJOR.MINOR.PATCH`) and
  **only changes when a human bumps it** — nothing auto-bumps it (no changesets, no
  semantic-release). A commit to `main` moves the git SHA; it does **not** move the
  version. Pre-1.0, bump **PATCH** for fixes/small features and **MINOR** for larger ones.
- **Bump as part of the PR that completes the work** — edit `version` in the same PR,
  don't leave it trailing.
- **The on-launch update is SHA-based, not version-based.** The `infinite` launcher
  fast-forwards the checkout on launch (once/24h, clean tree only, ff-only, offline-safe,
  skipped for `GROWTH_OS_CLI_NONINTERACTIVE=1`); opt out with `INFINITE_NO_AUTO_UPDATE=1`.
  `maybeNotifyUpdateAvailable` (in `apps/cli/src/index.ts`) prints the `⬆ Update
  available` notice. Both compare `origin/<branch>` vs local HEAD, so they react to every
  commit on `main`, not to releases.
- **Cutting a release (optional, manual).** A *release* is a blessed commit, marked by a
  **git tag** + a **GitHub Release** (the Release can host assets — e.g. `v0.1.0` hosts
  the GA4 `client_secret`). To cut one after the version-bump PR merges:
  ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Infinite-Labs-AI/infinite-os](https://github.com/Infinite-Labs-AI/infinite-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
