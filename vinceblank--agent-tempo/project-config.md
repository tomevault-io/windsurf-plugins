---
trigger: always_on
description: agent-tempo is an MCP server that enables multiple Claude Code sessions to coordinate via Temporal.
---

# CLAUDE.md

## What is this?

agent-tempo is an MCP server that enables multiple Claude Code sessions to coordinate via Temporal.

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
├── cli.ts             # CLI entry point (agent-tempo command)
├── daemon.ts          # Daemon entry point — runs Temporal workers as a detached background process
├── cli/
│   ├── commands.ts    # CLI command implementations (up, start, conduct, status, stop, …)
│   ├── command-center-command.ts # command-center subcommand — crash-proof; launches Pi mission-control board; sets AGENT_TEMPO_MISSION_CONTROL=1 (#729)
│   ├── config-command.ts # config subcommand (interactive + set/show) — crash-proof for show/set
│   ├── daemon.ts      # Daemon management utilities (start, stop, status, heartbeat, isDaemonRunning)
│   ├── daemon-command.ts # daemon subcommand handler — crash-proof, no Temporal deps
│   ├── dashboard-command.ts # dashboard subcommand — crash-proof; opens the web dashboard, optionally minting a QR-code pairing token (#340)
│   ├── ensure-infra.ts # shared infra bootstrap (`ensureInfra()`) — brings up Temporal, SAs, daemon; shared by `up` and the Pi extension `/ensemble-up` (#700 P1)
│   ├── dev-banner.ts  # [DEV MODE] banner formatter (ADR 0014 §5.4) — gate 4 production-safety line
│   ├── dev-mode-bootstrap.ts # pre-import side-effect: promotes top-level `--dev` flag to `CLAUDE_TEMPO_DEV_MODE=1` before any other module loads
│   ├── dev-verbs.ts   # dev-mode scriptable CLI verbs (#432) — shell-scriptable wrappers over MCP tools for E2E validation; stripped from production surface
│   ├── global-wrapper.ts # global wrapper auto-provisioning — writes `~/.agent-tempo/bin/agent-tempo` shell/cmd scripts; entrypoint pointer refreshed on every CLI boot; emits one-time PATH hint when bin dir is off PATH (#620)
│   ├── help-text.ts   # help output — crash-proof, no Temporal deps
│   ├── legacy-migration.ts # one-shot idempotent copy `~/.claude-tempo/` → `~/.agent-tempo/` on first v1.0 boot (PR-2 of rebrand)
│   ├── mcp.ts         # MCP server registration helpers (init, global vs project)
│   ├── output.ts      # Shared CLI output formatting helpers
│   ├── preflight.ts   # Environment preflight checks
│   ├── removed-verbs.ts # lookup table for the 10 CLI verbs removed in #288 — dispatches migration hints before loading Temporal surface
│   ├── resolve-ensemble.ts # canonical ensemble-name resolver — `--ensemble` flag > positional > env > 'default' (#685)
│   ├── sa-preflight.ts # search-attribute preflight — REQUIRED_SEARCH_ATTRIBUTES list (single source of truth), registerSearchAttribute, verifySearchAttributes, assertSearchAttributesOrExit
│   ├── scenarios-command.ts # scenarios subcommand (dev mode only) — list/show shipped YAML scenario library (ADR 0014 §4.8)
│   ├── startup.ts     # auto-provisioning bootstrap state machine (#289) — six-step idempotent sequence used by bare `agent-tempo` invocation
│   ├── upgrade-command.ts # upgrade subcommand — crash-proof; dynamic-imports Temporal only for active-session warning
│   └── upgrade-to-2-command.ts # upgrade-to-2 cutover verb (#785) — crash-proof; dynamic-imports the phase-engine + Temporal inside try/catch; --yes/--dry-run/--force-drain
├── adapters/
│   ├── README.md      # Adapter contract documentation
│   ├── index.ts       # Adapter registry bootstrap + barrel exports (mock registered iff isDevMode())
│   ├── base.ts        # BaseAttachment + SdkAttachment base classes (lifecycle skeleton)
│   ├── terminal-error.ts # Shared terminal-class error classifier for signal/query failures (#249)
│   ├── claude-code/   # InteractiveAttachment — Claude Code CLI adapter
│   ├── copilot/       # CopilotSdkAttachment — Copilot bridge adapter
│   ├── claude-api/    # ClaudeApiAttachment — headless adapter via Anthropic Messages API (#131)
│   ├── opencode/      # OpenCodeAttachment — headless multi-provider adapter via SST OpenCode subprocess (#449)
│   ├── pi/            # Headless Pi adapter — descriptor + spawn entry (Phase 3a). No BaseAttachment; the Pi extension singleton owns lifecycle (claim/heartbeat/tools/cue pump). `adapter.ts` is the process entry; `index.ts` is the registry descriptor.
│   ├── mock/          # MockAttachment — dev-mode-only SDK adapter (ADR 0014 PR-2). prepack strips dist/adapters/mock from npm tarball.
│   └── sdk/           # SDK-style adapter base (used by Copilot bridge and opencode). Key files: `idle-backoff.ts` (T0.2 IdleBackoff helper — base 2s, cap 30s/60s, reset-on-delivery), `doorbell-client.ts` (T1.1 DoorbellClient + WakeableSleep — reconnecting SSE consumer for `/doorbell`; ding→reset()+wake(); connected ceiling 60s; disconnected falls back to 30s T0.2 floor)
├── client/
│   ├── interface.ts   # TempoClient TypeScript interface and related types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinceblank/agent-tempo](https://github.com/vinceblank/agent-tempo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
