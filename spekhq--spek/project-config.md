---
trigger: always_on
description: Guidance for Claude Code working in this repo.
---

# CLAUDE.md

Guidance for Claude Code working in this repo.

## Project Overview

spek — an OpenSpec content viewer. Four delivery surfaces plus one CI helper:

- **Web** — local read-only Express + React SPA; pick a repo path in the UI and browse
- **VS Code** — Webview Panel over the current workspace's openspec
- **IntelliJ** — Tool Window + JCEF
- **Demo** — self-contained static HTML (`docs/demo.html`) embedding spek's own openspec, deployed to GitHub Pages
- **GitHub Action** (`spekhq/spek`) — generates an HTML snapshot + status badges in CI

## Repo

The repo is **`spekhq/spek`**; the npm scope is **`@spekjs`** (an org name ≠ npm scope is normal — don't "fix" the mismatch).

## Three things whose *names* are configuration — renaming any of them fails silently

Each is referenced by something outside the file that holds it, matched by exact string, with no error when the
match breaks. Rename only alongside updating the other side.

| Name | Referenced by | What a rename does |
|---|---|---|
| `ci.yml`'s job names — `Node gates`, `Kotlin gates`, `Composite action smoke test` | `master`'s branch protection (required status checks) | PRs sit at **pending forever**, waiting on a check that will never report. Reads like CI is down, not like a config error |
| `npm-publish.yml` (the filename) | npm's trusted publisher registration for both packages | The workflow still runs and still resolves the version difference; it fails only at authentication, naming no cause |
| `action.yml`'s build chain steps | nothing — it is the *absence* of a reference that bites | Outputs stay populated while the files behind them are empty or missing (see below) |

Branch protection on `master` requires those three checks with `strict: true` (a PR must be up to date before
merging), does **not** require reviews, and leaves `enforce_admins` off — deliberately, because the `release` skill
pushes the `npm version` commit straight to `master` and a locally-created commit can never have passed a required
check. Turning admin enforcement on breaks `/release`.

## action.yml: smoke-tested only — read before touching the build chain

CI runs a smoke job (`action-smoke` in `ci.yml`) that invokes the composite action against this repo with
`generate-badges: "true"` and asserts the `html-path` / `badges-path` outputs point at real, non-empty files. It
pins `spek-version: ${{ github.sha }}` — the action checks out `spekhq/spek` at that ref and builds from *that*
copy, so the default `master` would test master's implementation and go green on a PR that breaks the action.

**What it covers**: the action's own build chain produces output. That is the failure that shipped before.
**What it does not**: any input combination other than the one it runs (`repo-path`, `output-path`, `title`,
`spek-version` pinned to a tag), the generated HTML's *content*, and behavior on a consumer's repo layout. A
change to those still needs manual verification — a temporary `workflow_dispatch` workflow with
`uses: spekhq/spek@master`, asserting the outputs, then removed.

- Precedent: moving `@spekjs/ui`'s build from `prepare` (install-time) to `prepublishOnly` (publish-time) made the
  action's ui build **silently vanish** — it relied on `npm ci` triggering `prepare` to get ui dist. The Marketplace
  action was broken for a full day with nothing raising an alarm.
- `spek-version` defaults to `"master"` — a user who pins `@v1` still builds against master: master breaks → everyone
  breaks instantly.

## Tech Stack

- **`@spekjs/core`** — pure Node.js shared logic (scanner / tasks / types). Published to npm on its own version line;
  only runtime dep is `cross-spawn`. In-repo consumers resolve it locally via `"*"` workspaces, so development is
  independent of core's release cadence.
- **`@spekjs/ui`** — reusable visual components (`SpecGraph`, `ChangeTimeline`). Published to npm. **Purely
  presentational**: data in via props, selection out via callbacks; no router / adapter / CSS framework. Colors are 8
  `--spek-*` CSS variables (its own names, never the host's tokens). The web `/graph` and `/timeline` pages are thin
  shells (fetch / loading / navigation / theme).
- React 19 + Vite + TS + Tailwind v4; Express (REST); VS Code Webview + esbuild; IntelliJ Kotlin + JCEF + built-in
  server; react-markdown + remark-gfm (BDD highlighting); search = server-side full-text + Fuse.js; React Router v7
  (Web BrowserRouter / webview MemoryRouter).

## Project Structure

```
packages/
├── core/       # @spekjs/core — pure logic (scanner.ts, tasks.ts, artifacts.ts, schema-order.ts, schemas.ts,
│            #   schema-flow.ts, openspec-cli.ts, git-cache.ts, types.ts)
├── ui/         # @spekjs/ui — visual components (SpecGraph.tsx, timeline/*, theme.ts=color contract, styles.css)
├── web/        # @spekjs/web — server/ (Express API) + src/ (React SPA + API adapters)
├── vscode/     # spek-vscode — src/ (extension.ts, panel.ts, handler.ts) + webview/ (from web build:webview)
└── intellij/   # spek-intellij — src/main/kotlin/com/spek/intellij/ + resources/webview/ (from web build:intellij)
scripts/        # build-demo.ts, generate-badges.ts
docs/           # demo.html (Pages), prd.md, feature-ideas.md
.agents/skills/ # skill sources; .claude/skills/ are symlinks to them
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spekhq/spek](https://github.com/spekhq/spek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
