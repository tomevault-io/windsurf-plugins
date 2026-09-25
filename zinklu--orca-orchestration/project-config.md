---
trigger: always_on
description: Guidance for agents working **in this repo** (editing the viewer/tooling or the skill doc). For using the skill to build DAGs, read `skill/SKILL.md` and `README.md` instead. `CLAUDE.md` is a symlink to this file — edit here, don't replace the link.
---

# AGENTS.md

Guidance for agents working **in this repo** (editing the viewer/tooling or the skill doc). For using the skill to build DAGs, read `skill/SKILL.md` and `README.md` instead. `CLAUDE.md` is a symlink to this file — edit here, don't replace the link.

## Project shape

Two independent modules in one npm-workspaces repo (`workspaces: [server, web]`):

- **`skill/SKILL.md`** — a published agent skill (frontmatter + prose). Teaches an *external* agent to build an Orca orchestration task DAG via the `orca` CLI. No code; edit it like a doc, keep frontmatter intact.
- **`server/` + `web/`** — the `orca-dag` viewer: an Express API + React Flow SPA that visualizes a Run's DAG and runs a self-driven coordinator loop. Compiles to a single portable binary (`dist/orca-dag`).

This repo is a thin, heavily-commented wrapper over the `orca` CLI. Every Orca quirk is documented in `server/src/orca.ts` comments — **read them before touching orchestration code.**

## Commands

```bash
npm install
npm run dev            # BOTH server (:8787) + web (:5173) via concurrently; vite proxies /api → :8787
npm run dev:server     # tsx watch src/index.ts (server only)
npm run dev:web        # vite (web only)
npm run build          # web only: tsc -b && vite build → web/dist
npm run build:binary   # web build → embed as base64 in server/src/generated/webAssets.ts → bun --compile → dist/orca-dag
npm run build:npm      # web build → esbuild the server → dist-npm/ (the publishable `orca-dag` package; Node only, no Bun)
npm start              # server (tsx) against web/dist on disk; needs `npm run build` first for UI
```

- **No `lint`, `test`, or `typecheck` scripts exist.** Typecheck manually:
  - web: `npm run build -w web` (runs `tsc -b` first, fails fast on type errors)
  - server: `npx tsc -p server/tsconfig.json --noEmit`
- **`build:binary` requires `bun` on PATH** (not declared as a dependency). Cross-compile with `TARGET=bun-linux-x64 npm run build:binary`.
- Binary runtime env: `PORT` (8787), `NO_OPEN=1` (skip browser), `WORKSPACE_DIR` (overrides `active` worktree), `ORCA_WORKTREE` (default `active`).

## Distribution

Two artifacts ship from this repo, and **neither is an Orca plugin** — Orca's plugin panels are `srcdoc` iframes under `default-src 'none'; connect-src 'none'`, so a panel can't `fetch` the viewer's API at all, and `TabContentType` is a closed union with no registry for third-party panes. Orca also deliberately ships no scheduler ("A Run is a namespace and home inbox. It never schedules or places workers."), so an external coordinator like this one is the intended shape, not a workaround.

- **`npx orca-dag` is the whole install.** `server/src/skill.ts` writes `skill/SKILL.md` into every agent skills directory that exists under `$HOME` before the server listens, so the skill half and the viewer half arrive together. It must stay best-effort: never throw, never rewrite an unchanged file, and **never write through a symlinked skill directory** (that's the `ln -s "$PWD/skill"` recipe — following it would clobber someone's working copy). `--no-skill` / `ORCA_DAG_NO_SKILL=1` opts out.
- **`orca-dag uninstall` (`server/src/uninstall.ts`) is the mirror image and has to stay that way** — anything a future startup step writes outside the workspace must get a matching removal here, or the install becomes a one-way door. It shares `AGENT_SKILL_DIRS`/`SKILL_NAME` with `skill.ts` so the two can't drift. It unlinks symlinked skill dirs rather than following them, keeps `.orca-dag.config.json` unless `--purge` (it's real user work: harness/model picks and canvas layout), and closes every terminal whose title starts with `COORDINATOR_TITLE` — a crashed viewer otherwise leaves one bound to the Run, fencing the user's own agent.
- **Subcommands are dispatched at the top of `index.ts`**, before the express app is built, so `uninstall` and `--help` never bind a port, create an Orca terminal, or install the skill on their way out. Keep new subcommands in that block.
- **The skill** also installs through the community skills CLI straight from this repo: `npx skills add ZinkLu/Orca-Orchestration --skill orca-dag`. Discovery keys off `skill/SKILL.md`'s frontmatter — `scripts/check-skill.mjs` guards it in CI (don't shell out to `skills add . --list` there; it prompts when no agent is detected and hangs).
- **The binary embeds the skill too** (`SKILL_MD` in the generated `webAssets.ts`, next to `WEB_ASSETS`) — it has no package directory to read from, and a single downloaded file has to behave like the npm package.
- **The viewer** publishes as the `orca-dag` npm package (`npx orca-dag`), plus standalone Bun binaries attached to the GitHub release for people without Node. `scripts/build-npm.mjs` stages `dist-npm/`; the bundle lands at `dist/server/index.mjs` **on purpose** — `index.ts`'s disk fallback looks in `join(__dirname, "..", "..", "web", "dist")`, which only resolves inside the package at that depth. Moving either path breaks the SPA silently (API still answers, UI 404s), which is exactly what the CI smoke test checks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZinkLu/Orca-Orchestration](https://github.com/ZinkLu/Orca-Orchestration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
