---
trigger: always_on
description: Guidance for Claude Code working in this repository.
---

# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this is

**Swarm** — an open-source (Apache-2.0), local-first control plane for AI-agent development on *any* repository. One daemon watches every Claude Code session on the machine (via hooks, stream-json, and MCP), keeps a ledger of task claims, worktrees, runtime resources and verification gates, enforces rules as hook denials instead of prose, and streams everything to a dashboard. It is repo-agnostic: it must never require files inside a monitored repository; state lives in `~/.swarm/`.

Docs-first: start at [`docs/00-index.md`](docs/00-index.md); the user-facing surface (dashboard, CLI, MCP tools) is specified in `docs/08-interface.md`. Nothing is built from a doc still marked `draft` without flagging it. Open questions are `OQ-N` in `docs/07-open-questions.md`; decisions are recorded there as `> **Decision:**`, never inline in chat only. Task IDs in `docs/06-roadmap.md` are immutable; flip status the same turn work lands.

## Status

M0 + M0.8 + M0.9 done; M1 claims / resources / process registry (`swarm serve`/`proc`) / task source and M2 rules (incl. write rules) + Incidents view landed (2026-08-22, v0.4.0); M1 + M2 complete 2026-08-22 (auto-renew M1.2, handoffs + SessionStart context M1.3, gates M2.2); M3.1 `swarm run` (`daemon/src/runner.ts`) landed; M3.2 permission broker (`store.evaluateTool` shared with the hook) and M3.3 run-from-dashboard landed 2026-08-22; M3 complete. SQLite-backed (`~/.swarm/swarm.db`). Reads Claude Code **transcripts** for tokens/cost/reasoning, not just hooks — the dashboard at http://127.0.0.1:7777 shows every Claude Code session on the machine live (hooks installed user-wide via `swarm install`), with a project sidebar and Fleet / Board / PRs / Timeline / Spend / Stats views plus session detail. Config is `~/.swarm/config.toml` + optional `<repo>/.swarm.toml` (`core/src/config.ts`, docs/13); rules v2 (`core/src/rules.ts` `guardBash` for Bash, `guardWrite` for file tools / Bash cwd against claimed worktrees) return `ask`/`deny` on `PreToolUse` and record `incident.opened` (ack via `incident_acks`); task source is `core/src/tasks.ts`; process registry is `core/src/processes.ts` + `packages/cli/src/procs.ts`; runtime resources are `core/src/resources.ts`; PRs come from `gh`/`glab` via `daemon/src/forge.ts`. Dashboard is plain HTML/JS in `packages/web/public` (`index.html`, `app.js`, `viz.js`, `table.js` — the shared data-grid), served by the daemon. Three generated files sit beside them — `menus.js` (React island running `@react-fancy-menus/core`), `fm.css`, `icons.js` (Phosphor subset) — produced by `bun run build:web` (`packages/web/tools/build.ts`, runs on `postinstall`). Icons are **pixelarticons** everywhere (the subset is listed in `packages/web/tools/build.ts`); menus are described as data in `app.js` (`menuSpec`) and rendered by `src/menus.tsx`. **The dashboard is React** (M11, OQ-6 decision 2026-08-27): it lives in `packages/web/src`, is bundled by `Bun.build` into `public/dashboard.js` + `dashboard.css`, and serves `/`. The vanilla `app.js` is retired; `viz.js` and `table.js` remain in `public/` only because the **team** dashboard (`packages/team`) is still a vanilla page that loads them from there. Read `packages/web/src/README.md` before touching it — the conventions there (selectors return snapshot slices, columns at module scope, never wrap an `<svg>`, reuse the stylesheet's class names) each exist because breaking one produced a bug. `bun run check:classes` fails on a `className` with no rule behind it.

M10 (reclaim: build-output clearing + a stale threshold that fires) landed 2026-08-26. M4–M9 are complete (M5 + M8 finished 2026-08-24; **M9 Observatory finished 2026-08-26** — outcomes, stalls, waiting, context, MCP health, gate flakiness, hygiene, the graph engine and its five graphs, provenance, A/B trials, tool transitions, file heat, resource holding, security audit and rule effectiveness; the dashboard also has an error boundary that names version skew). Six agents observed behind `core/src/adapters` (Claude Code, Codex, Grok, Gemini CLI, Aider, opencode — the last two carry their own exact cost as `cost_fixed` turns that repricing skips). Teams (docs/14): `packages/team` is `swarm-teamd`, the self-hosted team daemon — **FSL-1.1-ALv2, the only non-Apache directory**, never imported by the free packages (daemon tests use stub `/t1/*` servers); the local daemon forwards audit events/spend/claims via an outbox (`[team]` config), `swarm login` does OIDC device-code auth + machine registration, cluster claims revoke local on conflict (worktree untouched), org policy is ed25519-signed and pinned at login, team budgets apply local warn/ask/stop, `swarm backup/restore`, `/t1/metrics`, `[notify] webhook`, `[models] allow`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ra3orblade/swarm](https://github.com/ra3orblade/swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
