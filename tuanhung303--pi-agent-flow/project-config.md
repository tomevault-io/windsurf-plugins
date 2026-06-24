---
trigger: always_on
description: > 🗺️ **This file is your index route.**
---

# pi-agent-flow — Project Notes

> 🗺️ **This file is your index route.**
> Think of it as the project's control panel — not a dry spec sheet, but an activation map. If you need to deploy, bump a version, debug a flow, or figure out which script to run, this file points you to the right door. Start here before wandering the codebase.
>
> 🌱 **Keep this index alive.**
> CLAUDE.md is a living document. When flows change, scripts move, or CI/CD steps get updated, this file must reflect reality. If you just changed something structural — added a workflow, renamed a script, tweaked a flow's tools — **update this file before you wrap up**. The next agent (or future you) will thank you. Don't leave them lost in the maze.

## Project Index

| Category | Files |
|----------|-------|
| **Docs** | [`docs/autonomous-pi-testing.md`](docs/autonomous-pi-testing.md) — scripted Pi sessions over PTY • [`docs/CONFIGURATION.md`](docs/CONFIGURATION.md) — configuration reference (flags, env vars, settings) • [`docs/FLOWS.md`](docs/FLOWS.md) — bundled flows, session modes, timeout behavior, flow loop & warp • [`docs/TOOLS.md`](docs/TOOLS.md) — tool reference (incl. field aliases) • [`docs/CUSTOM-FLOWS.md`](docs/CUSTOM-FLOWS.md) — custom flow creation and front-matter • [`docs/STRUCTURED-OUTPUT.md`](docs/STRUCTURED-OUTPUT.md) — structured JSON output schema • [`docs/NOTIFICATIONS.md`](docs/NOTIFICATIONS.md) — terminal and desktop notification config • [`docs/SNAPSHOT-INSTRUMENTATION.md`](docs/SNAPSHOT-INSTRUMENTATION.md) — dump capture, inspection, and verification guide • [`SHARED-CONTEXT-CORE2.md`](SHARED-CONTEXT-CORE2.md) — core-2 snapshot pipeline specification (verbatim-preserving) |
| **Dump Analysis** | [`docs/dump-artifacts/README.md`](docs/dump-artifacts/README.md) — catalog of representative dump files |
| **Workflows** | [`ci.yml`](.github/workflows/ci.yml) — lint + test on PR/push • [`bump-version.yml`](.github/workflows/bump-version.yml) — version bump → commit → tag → push → publish • [`publish.yml`](.github/workflows/publish.yml) — standalone manual fallback |
| **Scripts** | [`dev-start.sh`](scripts/dev-start.sh) — start `pi` with `PI_FLOW_DUMP_SNAPSHOT` preset • [`switch.sh`](scripts/switch.sh) — toggle local ↔ remote install • [`sync-dumps.sh`](scripts/sync-dumps.sh) — sync `/tmp` dumps into `dump-artifacts/` • [`example-autonomous-pi.expect`](scripts/example-autonomous-pi.expect) — PTY test harness template • [`./tmp/validate-context-pipeline.js`](./tmp/validate-context-pipeline.js) — synthetic context pipeline validator • [`./tmp/analyze-dump.js`](./tmp/analyze-dump.js) — real dump analyzer • [`./tmp/validate-trace-dispatch.js`](./tmp/validate-trace-dispatch.js) — trace dispatch smoke test |
| **Key Source** | `src/index.ts` — entrypoint • `src/flow/runner.ts` — flow fork runner and process management • `src/core2/snapshot.ts` — core-2 session snapshot builder (verbatim-preserving) • `src/flow/agents.ts` — bundled flow definitions & loading • `src/flow/cycle-guard.ts` — failover/retry guards and connection-error detection • `src/batch/index.ts` / `src/batch/` — unified file/batch tools • `src/tui/render.ts` — TUI rendering & animations • `src/snapshot/structured-output.ts` — JSON output validation & enrichment • `src/tools/trace.ts` — standalone verbatim read/check tool, zero required fields • `src/tools/trace-dispatch-prep.ts` — lenient dispatch normalization for trace/flow • `src/tools/web-tool.ts` — search & fetch • `src/tools/ask-user.ts` — interactive prompts • `src/config/config.ts` — settings resolution • `src/notify/notify.ts` — desktop/terminal notifications • `src/flow/loop.ts` — endless loop state management (enable, disable, reset, terminate, warp tracking) • `src/flow/loop-command.ts` — `/flow:loop` slash command (enable/disable/status/stop/reset) • `src/flow/auto-warp.ts` — auto-warp trigger when loop budget is exceeded • `src/flow/loop-templates.ts` — loop runtime prompt templates • `src/flow/warp.ts` — warp distillation and session creation • `src/flow/warp-overlay.ts` — glitching warp-status overlay with live token counter and Enter/Esc input handling |
| **Additional Source** | `src/flow/` — flow goal orchestration, continuation, loop, runner, executor, transitions, depth config, session modes, and settings commands • `src/steering/` — steering hint injection, sliding prompts, and tool utilities • `src/types/` — shared TypeScript types for flow execution, output, and UI • `src/batch/` — batch operation engine, rendering, fuzzy editing, symbol extraction • `src/batch/normalize.ts` — shared batch/bash op normalization (extracted from batch/index.ts and batch-bash.ts) • `src/config/` — TUI-safe logging and settings resolution • `src/tui/` — color themes, render utilities, single-select layout, scramble animation • `src/snapshot/` — CLI arg inheritance, runner event parsing • `src/tools/` — timed bash wrapper with deadline awareness • `src/notify/` — notification state tracking |

## CI/CD

Publishing is **fully automated** via GitHub Actions.

### Strict Rules
- **Never run `npm publish` locally.** Always use CI.
- **Never run `npm version` locally.** The Release workflow handles tagging.

### Publish Flow

When the user asks to publish:

1. Merge feature branch to `main` and push.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuanhung303/pi-agent-flow](https://github.com/tuanhung303/pi-agent-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
