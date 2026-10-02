---
trigger: always_on
description: A cartoon first-person 3D office (React Three Fiber) over a Node orchestrator that runs one Claude Code session
---

# cubefarm

A cartoon first-person 3D office (React Three Fiber) over a Node orchestrator that runs one Claude Code session
(Claude Agent SDK) per developer, QA tester and the CEO, each in its own git worktree, working through GitHub issues.
This is the app the company runs on: a live office is running from this repo right now. README.md is the quick start; docs/how-it-works.md and CONTRIBUTING.md have the details.
It ships on npm as `cubefarm` (`npx cubefarm`); it used to be called Office Swarm.

## SAFETY (read first)

- The live office runs from `C:\Projects\office-swarm` on this machine, on ports 4317 (server) and 5317 (Vite),
  with its state in `~/.cubefarm`. Never edit or run anything there, never read or write `~/.cubefarm` directly, and
  never use ports 4317 or 5317.
- Test only in demo mode (fake GitHub, fake agents, no Claude usage), with an isolated `SWARM_HOME` and your reserved
  `SWARM_PORT` (from your job instructions):

  ```bash
  npm install
  npm run build
  SWARM_HOME="$PWD/.swarm-home" SWARM_PORT=<your port> node --import tsx server/index.ts --demo
  ```
  ```powershell
  npm install; npm run build
  $env:SWARM_HOME="$PWD\.swarm-home"; $env:SWARM_PORT="<your port>"; node --import tsx server/index.ts --demo
  ```
  Then open `http://localhost:<your port>` (the server serves the built `dist/`). The startup banner must say
  `DEMO MODE` and print a `state:` path inside your `SWARM_HOME`. Stop it when done; don't commit `.swarm-home`.
- Never real mode (no `--demo`), and never `npm run dev` / `npm run demo` / `npm start` / `npx cubefarm`: they default
  to 4317, and `dev`/`demo` default Vite to 5317. The first three run `scripts/office.mjs`, the launcher, which also
  updates its own folder (git fetch + merge, npm install, build) when the office or you ask for it (`u`). Run it
  only in a throwaway clone outside the live office, with `--demo` and your `SWARM_HOME`, `SWARM_PORT` and
  `SWARM_CLIENT_PORT`.
- Agents are ordinary coding-agent CLI sessions on the manager's own setup, unsandboxed, by the manager's choice. The
  office's workflow rules (no pushes to the default branch, no merging, QA leaves GitHub alone) live in their prompts
  and instructions, not in enforcement: don't add hooks, permission rules or sandboxes that refuse tool calls.
  `ANTHROPIC_*` / `CLAUDE_*` are stripped from agent and preview env.

## Scripts

| Command | What it does |
| --- | --- |
| `npm run typecheck` | `tsc --noEmit` over client, server, shared, the `.ts` in scripts and the configs |
| `npm test` | Vitest, once (`npm run test:watch` to re-run on edits) |
| `npm run build` | typecheck, `vite build` to `dist/`, then the server bundled into `dist-server/` (`scripts/build-server.mjs`) |
| `npm run test:e2e` | Playwright (`e2e/`, `playwright.config.ts`): builds, boots a demo office on `E2E_PORT` (default 4399; set it to your reserved port) with a temp `SWARM_HOME`, and smoke-tests it in headless Chromium |
| `node scripts/smoke-package.mjs` | after a build: packs the npm package, installs it into a temp folder and boots its demo |
| `node --import tsx server/index.ts --demo` | a demo office (see SAFETY for the env it needs) |

CI (`.github/workflows/ci.yml`): Node 24 on `ubuntu-latest` and `windows-latest`, `npm ci` → `typecheck` → `test` →
`build` → package smoke test, for every PR and push to `main`, with a throwaway `SWARM_HOME` and `SWARM_PORT=0`; a separate `e2e`
job on `ubuntu-latest` runs `npm run test:e2e`. All three must pass locally
before you open a PR.

## Code map

Server (`server/`, Node + Express 5 + ws, run by tsx in development; esbuild bundles it into `dist-server/` for npm):
- `index.ts`: entry; picks the real or demo backend, REST routes under `/api`, the `/ws` and `/ws/term` websockets, serves `dist/`, shutdown.
- `config.ts`: `SWARM_PORT` (default 4317), `SWARM_HOME` (default `~/.cubefarm`), `--demo`, state file, intervals,
  the default projects folder.
- `swarm.ts`: the orchestrator. Floors, agents, scheduling/auto-assign, dev → QA → fix → merge loop, dev and QA
  prompts, CEO job queue, phone messages, persistence (`state.json` / `demo-state.json`), websocket fan-out.
- `agentRunner.ts`: one Agent SDK session; options, env stripping, Playwright MCP, and turning the SDK stream into
  terminal lines.
- `cliRunner.ts`: the terminal runtime (the default): one agent as the real CLI in a node-pty, same session contract
  as `agentRunner.ts`. Claude Code reports through HTTP hooks (`POST /api/hooks/:token`; PreToolUse approves every
  call); Codex reports turn endings (notify) and, once the manager trusts them, its steps (`-c hooks.*`); OpenCode
  only turn endings. Codex/OpenCode screenshots are collected from the session's Playwright output folder. Esc in a
  terminal ends the session as `interrupted`. The CEO's office tools are served over MCP (`/api/mcp/:token`).
- `ptyHost.ts`: the terminal keeper, a detached process of its own (`launch` re-spawns it outside the office's process
  tree) holding the CLIs' pseudo-terminals and relaying their hooks, so agents keep working through office restarts.
  `ptyClient.ts` is the office's side (start/connect, spawn, adopt after a restart, local fallback); `ptyProtocol.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonvanzyl/cubefarm](https://github.com/leonvanzyl/cubefarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
