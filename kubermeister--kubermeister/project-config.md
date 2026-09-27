---
trigger: always_on
description: Desktop Kubernetes client (Electron).
---

# Kubermeister

Desktop Kubernetes client (Electron).

This file is the project's agent instructions. `CLAUDE.md` only imports it, so edits go here.

## Commands

- `npm run dev` starts Electron with Vite HMR. `npm run build` writes `out/`.
- Node 24 and npm 11.19 or newer are required (`engines` + `engine-strict`). CI runs the same
  versions; older npm silently drops optional lockfile entries and breaks `npm ci`.
- Since Electron 42 the npm package no longer downloads its binary on install; the `postinstall`
  script runs Electron's installer so `node_modules/electron/dist` exists for electron-vite dev
  and for the license notices packaging copies, then enables the git hooks. After an install with
  `--ignore-scripts`, run `node node_modules/electron/install.js` by hand.
- `npm run package` builds the current OS's installers into `release/` (`package:dir` for a fast
  unpacked bundle). The artifact name pattern in `electron-builder.yml` is load-bearing for the
  release workflows; change both together.
- After every change run `npm run lint`, `npm run typecheck`, `npm run format`, and `npm run test`.
  ESLint does not type-check, and Prettier covers the whole repo including Markdown and JSON.

## Git workflow

- **Never commit on `main`.** Create a branch first: `type/short-slug` (kebab-case, 2 to 4 words,
  no issue numbers, no usernames). Example: `feat/ipc-bridge`. The `pre-commit` hook refuses a
  commit while HEAD is `main`.
- Every change lands as a **squash-merged PR**. The PR title is the resulting commit header on
  `main` and the PR body is its body, so both follow the commit rules below.
- Open PRs with `gh pr create`. Never merge; the user merges.
- The PR body becomes the commit body on `main` and GitHub re-wraps it at 72 columns: write each
  paragraph as one unwrapped line. GitHub appends ` (#N)` to the title: keep PR titles at 66
  characters or fewer.
- The hooks live in `.githooks` (`commit-msg`, `pre-commit`). `npm install` enables them through
  `scripts/enable-hooks.mjs` in `postinstall`, which sets `core.hooksPath` only when it is unset, so
  a path the developer chose is left alone. By hand: `git config core.hooksPath .githooks`. Hook
  and script tests live in `tests/unit/repo`.
- CI runs on pull requests against `main` only, so a PR stacked on another branch gets nothing but
  the title check until it is retargeted.
- Retarget every child to `main` before merging its parent, because GitHub closes a PR whose base
  branch is deleted and never reopens it. Push after retargeting, since a base change alone starts
  no checks.

### Issues are the plan

- There is no board: the open milestones (one per upcoming minor version) and their issues are what
  is going to happen. A session resumes from `gh issue list` filtered by milestone or by the
  `ready` label.
- **An issue has a milestone.** Something worth keeping but not scheduled is an idea, and ideas live
  in the repository's **Ideas** discussion category, not in the issue list, which would otherwise
  stop being the plan. `gh discussion list` reads them beside `gh issue list`.
- An idea becomes an issue on the day it gets a milestone, filed the same way as any other work; an
  issue that turns out to have no release it belongs to moves the other way and closes as not
  planned pointing at its discussion.
- **There is no label for "not now."** A discussion is the whole mechanism, so nothing marks an
  open issue as set aside: it is either going to happen or it is a discussion. Moving one is three
  steps — create the discussion in Ideas carrying the issue's own words, comment the link on the
  issue, close it as not planned keeping its labels.
- A bug found in real use gets an issue before its fix.
- A PR that resolves an issue ends its body with the one-line paragraph `Closes #N.` (a sentence,
  not a trailer).
- File work with `gh issue create` giving `--type`, one `area:` label, a `size:` label and the
  milestone.
- The labels are defined in `.github/labels.yml` and change only through that file, which the
  `labels.yml` workflow syncs on merge.
- PR labels for the release notes come from the title's type (`pr-labels.yml`,
  `.github/release.yml`); never set them by hand.

### Commit messages (Conventional Commits 1.0)

```
type(scope): subject

Body: why the change is needed, what a reader of the history cannot learn from the diff.
```

- **type**: `feat` `fix` `perf` `refactor` `docs` `test` `chore` `ci` `build` `style` `revert`.
- **scope** (required): `repo` `main` `preload` `renderer` `shared` `ipc` `k8s` `charts` `build`
  `ci` `deps` `docs` `release`. The list lives in three places — here, `.githooks/commit-msg` and
  `.github/workflows/pr-title.yml`, which gates the PR title the squash commit takes — so a new
  area adds its scope to all three in the same change, and `tests/unit/repo/githooks.test.ts`
  fails when they drift apart.
- **subject**: lowercase, imperative, no trailing period, whole header 72 characters or fewer.
  Proper nouns that need capitals go in the body.
- Breaking change: `!` after the scope, e.g. `feat(ipc)!: rename stream channels`.
- **No trailers.** No `Co-Authored-By`, no `Signed-off-by`, nothing after the body.

### The changelog


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubermeister/kubermeister](https://github.com/kubermeister/kubermeister) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
