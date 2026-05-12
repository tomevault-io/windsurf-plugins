---
trigger: always_on
description: claude-tempo is an MCP server that enables multiple Claude Code sessions to coordinate via Temporal.
---

# CLAUDE.md

## What is this?

claude-tempo is an MCP server that enables multiple Claude Code sessions to coordinate via Temporal.

## Tech Stack

- **Runtime**: Node.js 20+ with TypeScript
- **MCP**: `@modelcontextprotocol/sdk` (stdio transport)
- **Temporal**: `@temporalio/client`, `@temporalio/worker`, `@temporalio/workflow`, `@temporalio/activity`
- **croner** — cron expression parsing and next-fire computation (used by `schedule` tool)
- **yaml**, **zod** — lineup parsing and schema validation

## Project Structure

```
src/
├── server.ts          # MCP server entry point
├── cli.ts             # CLI entry point (claude-tempo command)
├── daemon.ts          # Daemon entry point — runs Temporal workers as a detached background process
├── cli/
│   ├── commands.ts    # CLI command implementations (up, start, conduct, status, stop, …)
│   ├── config-command.ts # config subcommand (interactive + set/show) — crash-proof for show/set
│   ├── daemon.ts      # Daemon management utilities (start, stop, status, heartbeat, isDaemonRunning)
│   ├── daemon-command.ts # daemon subcommand handler — crash-proof, no Temporal deps
│   ├── dashboard-command.ts # dashboard subcommand — crash-proof; opens the web dashboard, optionally minting a QR-code pairing token (#340)
│   ├── dev-banner.ts  # [DEV MODE] banner formatter (ADR 0014 §5.4) — gate 4 production-safety line
│   ├── dev-mode-bootstrap.ts # pre-import side-effect: promotes top-level `--dev` flag to `CLAUDE_TEMPO_DEV_MODE=1` before any other module loads
│   ├── help-text.ts   # help output — crash-proof, no Temporal deps
│   ├── mcp.ts         # MCP server registration helpers (init, global vs project)
│   ├── output.ts      # Shared CLI output formatting helpers
│   ├── preflight.ts   # Environment preflight checks
│   ├── removed-verbs.ts # lookup table for the 10 CLI verbs removed in #288 — dispatches migration hints before loading Temporal surface
│   ├── scenarios-command.ts # scenarios subcommand (dev mode only) — list/show shipped YAML scenario library (ADR 0014 §4.8)
│   ├── startup.ts     # auto-provisioning bootstrap state machine (#289) — six-step idempotent sequence used by bare `claude-tempo` invocation
│   └── upgrade-command.ts # upgrade subcommand — crash-proof; dynamic-imports Temporal only for active-session warning
├── adapters/
│   ├── README.md      # Adapter contract documentation
│   ├── index.ts       # Adapter registry bootstrap + barrel exports (mock registered iff isDevMode())
│   ├── base.ts        # BaseAttachment + SdkAttachment base classes (lifecycle skeleton)
│   ├── terminal-error.ts # Shared terminal-class error classifier for signal/query failures (#249)
│   ├── claude-code/   # InteractiveAttachment — Claude Code CLI adapter
│   ├── copilot/       # CopilotSdkAttachment — Copilot bridge adapter
│   ├── claude-api/    # ClaudeApiAttachment — headless adapter via Anthropic Messages API (#131)
│   ├── opencode/      # OpenCodeAttachment — headless multi-provider adapter via SST OpenCode subprocess (#449)
│   ├── mock/          # MockAttachment — dev-mode-only SDK adapter (ADR 0014 PR-2). prepack strips dist/adapters/mock from npm tarball.
│   └── sdk/           # SDK-style adapter base (used by Copilot bridge and opencode)
├── client/
│   ├── interface.ts   # TempoClient TypeScript interface and related types
│   └── index.ts       # TempoClient factory implementation and barrel re-exports
├── worker.ts          # Temporal worker setup (used by daemon only)
├── connection.ts      # Temporal connection factory (shared by server + CLI)
├── constants.ts       # Shared string constants (ensemble ready banner/directive, etc.)
├── spawn.ts           # Cross-platform process spawning helpers
├── workflows/
│   ├── session.ts     # claude-session workflow
│   ├── scheduler.ts   # durable scheduler workflow (one per ensemble)
│   ├── maestro.ts     # Maestro workflows — per-ensemble hub and global hub
│   ├── attachment-math.ts # Pure CAN-boundary lease-extension helper (no Temporal imports)
│   ├── maestro-signals.ts / scheduler-signals.ts / signals.ts   # Signal/query/update type defs
│   └── index.ts       # Workflow re-exports for worker bundle
├── activities/
│   ├── outbox.ts         # Outbox delivery activities (cue, report, recruit, release, spawn)
│   ├── maestro.ts        # Maestro activities
│   ├── hard-terminate.ts # Per-host process kill activity (used by destroy when attached)
│   ├── resolve.ts        # Session resolver shared by outbox + schedule-fire activities
│   └── schedule-fire.ts
├── http/              # Daemon HTTP/SSE event source (#94/#95)
│   ├── server.ts      # Express-style HTTP server — snapshot + streaming endpoints
│   ├── event-bus.ts   # In-process EnsembleEventBus (fanout to SSE clients)
│   ├── event-types.ts # TempoEvent / ClusterEvent wire type definitions
│   ├── sse-handler.ts # SSE response lifecycle (ring-buffer replay, gap detection, backpressure)
│   ├── ring-buffer.ts # Fixed-size event ring buffer (256 events) for Last-Event-ID replay
│   ├── snapshot.ts    # On-demand ensemble state snapshot (prelude + poll)
│   ├── aggregate.ts   # AggregateRunner — wires bus + snapshot + HTTP server startup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinceblank/claude-tempo](https://github.com/vinceblank/claude-tempo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
