---
trigger: always_on
description: This file gives external contributors and optional AI coding tools the small
---

# AGENTS.md - Goshtoso

This file gives external contributors and optional AI coding tools the small
set of repo rules that matter most. `CLAUDE.md` is a symlink to this file for
harnesses that look for that name. Local harness-specific directories such as
`.codex/` and `docs/superpowers/` are intentionally ignored and not part of the
public source tree. The curated `.agents/skills/` entries for Alpine.js, HTMX,
Tailwind CSS, and templ stay tracked as optional core-stack references. A small
generated compatibility reference remains under `.claude/skills/using-goshtoso/`
because CI checks it after component API changes.

## Project

Goshtoso is a Go UI component library built with templ, Tailwind CSS v4, HTMX,
and Alpine.js. The root module is the publishable library
`github.com/araihu/goshtoso`; `site/` is a separate module for the demo site,
examples, and E2E tests.

Alpine.js, HTMX, and the htmx SSE extension are bundled locally under
`assets/js/runtime/<module>/<version>/`; pinned versions live in
`assets/js/runtime/versions.json`. Runtime assets are served through
`assets.Handler()`, so page loads should not depend on a CDN. Regenerate runtime
URL constants with `go run ./cmd/vendorgen` and update pinned JS with
`just vendor-js`.

## Commands

```bash
# One-time local workspace for editing the library and site together.
go work init . ./site

# Regenerate templ output after editing .templ files.
templ generate

# Rebuild generated CSS after editing CSS or introducing Tailwind utilities.
just css

# Build the demo server.
go build -o bin/server ./site/cmd/server

# Unit tests.
go test ./... -count=1
cd site && go test $(go list ./... | grep -v /tests/e2e) -count=1

# Full E2E suite.
go test ./site/tests/e2e/... -count=1 -timeout 15m

# Specific E2E test.
go test ./site/tests/e2e/... -count=1 -timeout 5m -run TestDropdown
```

## Two Modules

- Root: component library. Keep dependencies slim and never import `site/`.
- `site/`: demo website, example apps, server, Playwright E2E tests.

`go.work` is gitignored. The site module pins a released library version for
fresh clones; CI creates a temporary workspace so it builds against the in-repo
library at the current commit.

## Worktree Isolation (required)

Every unit of work — a feature, an example app, a bugfix, a coverage pass — MUST
happen in its own dedicated git worktree branched from `origin/main`. Never edit,
build, or commit feature work in the shared primary checkout, and never reuse one
worktree for two unrelated tasks. Concurrent agents sharing a single working tree
produce entangled diffs (one task's changes leak into another's commit, generated
files collide, and a clean per-feature PR becomes impossible to assemble).

Start every task with:

```bash
git fetch origin
git worktree add -b <type>/<short-slug> /tmp/gs-<short-slug> origin/main
cd /tmp/gs-<short-slug>
go work init . ./site   # go.work is gitignored; recreate it per worktree
```

Rules:

- Branch from `origin/main` only — never from the current branch or another
  feature branch. Run `git fetch origin` first so the base is up to date.
- One worktree per task. If two agents work in parallel, they get two worktrees.
- Do all edits, `templ generate`, `just css`, builds, tests, and the commit
  inside that worktree. The primary checkout stays clean.
- Remove the worktree after the branch merges: `git worktree remove <path>`.

When you (or the harness) hand a task to a sub-agent, the worktree boundary is
how that work stays mergeable. Skipping it is what creates an unrecoverable
tangle of mixed, half-finished changes across shared files.

## Generated Files

Never hand-edit generated files:

- `*_templ.go` - regenerate with `templ generate`
- `assets/styles.css` - regenerate with `just css`
- `assets/goshtoso-theme.css` and vendored runtime constants - regenerate with
  the matching cmd/just targets

When resolving merge conflicts, resolve source `.templ` files first, then run
`templ generate`; do not hand-resolve generated templ output. If
`templ generate` reports no updates but rendering looks stale, remove the
affected generated file and regenerate.

## Component Workflow

Components live under `components/<name>/` with `types.go`, `<name>.templ`, and
generated `<name>_templ.go`.

Public component config fields should follow
[`docs/COMPONENT_API_NAMING.md`](docs/COMPONENT_API_NAMING.md). Prefer
target-specific names for shared hooks (`RootClass`, `InputAttrs`, `HTMX`,
`Alpine`) and role-specific labels (`Label`, `ActionLabel`, `HelperText`)
instead of reusing generic names with different effects.

When adding or changing a component:

1. Update the component source in `components/<name>/`.
2. Update or add the demo page under `site/internal/pages/demo/components/`.
3. Register the page in the demo registry and component catalog.
4. Register any component-specific HTTP endpoints in `site/internal/server/server.go`.
5. Add focused E2E coverage under `site/tests/e2e/`.
6. If `types.go` or entry points changed, run `go run ./cmd/skillgen`.
7. Run `templ generate`, `just css` when needed, and the relevant tests.

Demo pages should use one preview and one code block per variant. Keep variant

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [araihu/goshtoso](https://github.com/araihu/goshtoso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
