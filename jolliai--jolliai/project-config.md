---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Critical rules

These rules affect whether a change can ship at all. They override any other guidance below.

- **DCO sign-off on every commit** — `git commit -s`. CI rejects PRs without `Signed-off-by:`.
- **No `Co-Authored-By: Claude …` trailer or `🤖 Generated with …` footer.** Commit messages and PR descriptions stay human-authored; only the DCO `Signed-off-by:` trailer belongs there.
- **`npm run all` must pass before commit** (clean → build → lint → test). CI runs the same chain.
- **Do not regress CLI test coverage.** New code under `cli/src/` is held to 97% statements / 96% branches / 97% functions / 97% lines (see [`cli/vite.config.ts`](cli/vite.config.ts)).
- **Three implementations of the API key parser stay in lockstep.** `parseJolliApiKey` / `assertJolliOriginAllowed` live in [`cli/src/core/JolliApiUtils.ts`](cli/src/core/JolliApiUtils.ts), are bundled into the VS Code extension verbatim, and have a Kotlin port in `intellij/`. Updating one without the others is a known-bad pattern.
- **Cross-package imports in `vscode/src/**` are intentional.** Paths like `../../../cli/src/core/JolliApiUtils.js` resolve at esbuild bundle time. Don't refactor them into a published-package import — VS Code currently bundles the CLI inline.
- **Worktree-aware code only.** Hooks, summary storage, and lock files must work across `git worktree` checkouts. Don't assume a single working tree.
- **Suspected vulnerabilities go through [`SECURITY.md`](SECURITY.md)**, not public issues or PRs.
- **Workflow injection hygiene.** Inside `.github/workflows/*.yaml`, any `${{ … }}` expression derived from a user-controlled context — `github.event.*`, `inputs.*`, `github.head_ref`, the commit author/message fields, etc. — must be funnelled through `env:` before reaching a `run:` block or the `with:` of an untrusted action. Direct interpolation is the injection pattern. The existing publish workflows already follow this for both `inputs.tag` and `github.event.release.tag_name`.

## Repository layout

Monorepo with three deliverables that share the same product model and storage (a git orphan branch `jollimemory/summaries/v3`):

- `cli/` — `@jolli.ai/cli` (npm workspace, Node 22.5+, ESM, Vite multi-entry lib build). Standalone command-line tool plus all git/agent hook scripts.
- `vscode/` — `jollimemory-vscode` extension (npm workspace, esbuild → CJS). Bundles the CLI and hook scripts into its own `dist/` so it has **no dependency on a global CLI install**.
- `intellij/` — separate Gradle/Kotlin project (JDK 21). Independent build, but installs the same git/agent hooks and writes to the same shared state: the orphan branch, the machine-global `~/.jolli/jollimemory/` (config + hook entry scripts), and the per-project `<projectDir>/.jolli/jollimemory/` (sessions, cursors, queue, …).

Root `package.json` only coordinates the two npm workspaces; it does not touch `intellij/`. Use `.nvmrc` (currently `24.10.0`) for the Node version.

**Names you'll see (all refer to the same product):**

- **`jollimemory`** — product name; orphan branch prefix (`jollimemory/summaries/v3`).
- **`jolliai`** — current GitHub org / repo namespace (`github.com/jolliai/jolliai`).
- **`@jolli.ai/cli`** — npm scope and package name for the CLI workspace.
- **`jollimemory-vscode`** — `package.json` `name` of the VS Code extension. Root npm scripts reference it via the **workspace path** `vscode` (e.g. `npm run test -w vscode`), not by package name.

## Common commands

From the repo root (npm workspace; coordinates `cli` + `vscode`):

```bash
npm install            # install workspaces
npm run build          # build cli, then vscode (vscode esbuild bundle inlines cli/src/**)
npm run typecheck      # tsc --noEmit in both
npm run lint           # biome check --error-on-warnings (tab indent, 120 line width)
npm run lint:fix       # biome check --write
npm run test           # vitest run --coverage in both (cli enforces 97% threshold)
npm run all            # clean → build → lint → test (use this before committing)
```

Per-workspace variants exist for every script: `npm run build:cli`, `npm run test:vscode`, `npm run typecheck:cli`, etc.

Running a single test (Vitest):

```bash
# cli — vitest is the test script directly
npm run test -w @jolli.ai/cli -- src/core/SummaryStore.test.ts -t "merges children"

# vscode — same flags, but tests are launched via scripts/run-vitest.mjs
npm run test:vscode -- src/services/JolliPushService.test.ts -t "rejects http"
```

Iterating on the CLI without rebuilding: `npm run cli -- <command>` (uses `tsx` on the source). For end-to-end testing of the actual built artifact, do `cd cli && npm run build && npm install -g .` once — the global symlink keeps pointing to local `dist/`, so subsequent `npm run build` runs are picked up immediately by the global `jolli` binary.

VS Code extension iteration: `cd vscode && npm run deploy` bumps patch version → builds → packages → installs the VSIX. Then **Developer: Reload Window** in VS Code. If you also changed `cli/src/**`, run `cd cli && npm run build` first because the extension bundles the CLI at build time.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolliai/jolliai](https://github.com/jolliai/jolliai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
