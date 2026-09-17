---
trigger: always_on
description: Reviewer is a macOS Electron app for reading code reviews an agent wrote. `rvw` — a Stricli CLI
---

# CLAUDE.md

Reviewer is a macOS Electron app for reading code reviews an agent wrote. `rvw` — a Stricli CLI
bundled beside the app — takes a finished review as JSON, proves every anchor places against the
real diff, and writes a `.reviewer.json` artifact the app renders as an ordered tour. `README.md`
is the user's view of that. This file is for whoever edits the code.

Every line in this repository was written by an agent and no human has read it. That is the
operating constraint, not a disclaimer: there is no reviewer downstream to catch a convention you
dropped, so a convention only counts if the compiler, a test, or a comment explaining *why*
enforces it — in that order of preference. Prefer making a mistake impossible over writing down
that it is a mistake.

## The comments are the design record

Module headers here say why the code is shaped the way it is, what was tried instead, and which
failure the shape prevents. They are the only record of that reasoning — there are no design docs,
no PR threads, no commit-message essays. Concretely load-bearing examples:

- `tsconfig.shared.json`'s header is the whole argument for the node-free boundary; delete it and
  the next agent "simplifies" the project away.
- `src/renderer/src/index.css`'s glass block documents two Chromium/Electron traps in
  `backdrop-filter`. One of them fires *only* in a packaged build — Lightning CSS runs on build,
  not under `bun dev` — where nobody is iterating.
- `src/shared/diff/walk.ts` explains why two hunk-geometry models deliberately coexist and where
  they are allowed to disagree.
- `.oxlintrc.json` turns off `eslint/max-lines`, `max-lines-per-function` and `no-inline-comments`
  precisely so this style is legal, and every other rule it disables carries the reason it is off.

So: a refactor moves prose with the code it describes. Delete a comment only when the thing it
describes is gone. New non-obvious code gets the same treatment — the reason, the rejected
alternative, the failure prevented. Markdown is excluded from `oxfmt` (`.prettierignore`, which
oxfmt reads by default along with `.gitignore`) because the formatter is non-idempotent on it.

## Layout, and who may import what

| Path | What it is |
|---|---|
| `src/shared/` | The domain: zod contracts and pure functions. Renderer-safe, **node-free**. |
| `src/shared/node/` | The node half of shared — spawn argv, env hardening, `~/.rvw` paths. Main + CLI only. |
| `src/main/` | The Electron main process: window, menu, IPC handlers, git, the session/settings/progress stores. |
| `src/preload/` | The sandboxed bridge. Bundles `shared/ipc.ts` and nothing heavier. |
| `src/renderer/src/` | The React app: `components/`, `lib/` (pure helpers + hooks), `stores/` (zustand), `dev/` (the preview harness). |
| `src/tools/` | Review tooling that is pure and I/O-free: schema emission, validation, artifact assembly, coverage. Shared by the CLI *and* the renderer. |
| `cli/` | `rvw`: the Stricli app, its six verbs, and the effectful shell around them. |
| `design/` | The palette. `globals.css` is consumed; the rest is provenance — see `design/README.md`. |
| `skills/` | The agent-facing review skill `rvw skills` points at. Shipped as `extraResources`. |
| `scripts/` | `reset-state.mjs` (back to a first launch), `gen-icon.mjs`, `check-package.mjs` (asserts on the packaged artifact), `pack-cli.mjs` + `install-cli.sh` (`rvw` without the app, for Linux). |

The edges that actually exist, and are the ones to keep:

- **renderer → `src/shared/` (never `src/shared/node/`) and `src/tools/`.** `lib/coverage.ts` and
  `lib/overview.ts` import `tools/review-coverage`, which is why `src/tools/**` is in the web
  project too. Nothing in the renderer imports main, preload internals, or `cli/`.
- **main → `src/shared/` including `src/shared/node/`.**
- **preload → `src/shared/ipc.ts` only,** and that module takes `IpcContract` from
  `ipc-schemas.ts` with `import type` so it erases. The sandboxed preload must not pull zod in.
- **cli → `src/shared/`, `src/shared/node/`, `src/tools/`,** plus exactly one main file:
  `src/main/review/guard.ts`, imported only by `cli/exit-gate.test.ts`, whose claim is that the
  *app's* importer accepts what the CLI emits.
- **Nothing imports the renderer.**

### What enforces it

Four tsconfig projects, all with the same strictness flags. They are the enforcement, not
documentation of it — the first three below are `composite`, so an import across a boundary is
`TS6307` ("not listed within the file list of project") at typecheck time rather than a Vite build
error or a runtime crash in the window.

- `tsconfig.node.json` — main + preload + shared, plus the root `electron.vite.config.*` and
  `vitest.config.*`, which have to be typechecked somewhere.
- `tsconfig.web.json` — renderer + shared + tools (and `src/preload/*.d.ts`, which is the `Window`
  augmentation, not preload code), with everything under `src/shared/node/` **excluded**. That
  exclusion is one direction of the boundary.
- `tsconfig.shared.json` — everything under `src/shared/` except `node/` and the tests, with
  `"types": []`. Deliberately *not* composite — it needs no project graph, because its check is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alxnddr/reviewer](https://github.com/alxnddr/reviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
