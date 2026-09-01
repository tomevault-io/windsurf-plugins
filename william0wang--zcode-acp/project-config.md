---
trigger: always_on
description: **zcode-acp-server** — a Node.js bridge that connects the ZCode agent backend
---

# Agent Instructions

## Project overview

**zcode-acp-server** — a Node.js bridge that connects the ZCode agent backend
(`zcode app-server --stdio`) to any ACP-compatible editor (Zed, JetBrains, …)
via JSON-RPC over stdio. Translates ACP protocol requests into ZCode session
methods and streams events back as ACP `session/update` notifications.

## Commands

| Task                        | Command                               |
| --------------------------- | ------------------------------------- |
| Build                       | `pnpm build`                          |
| Typecheck                   | `pnpm typecheck`                      |
| Test (all)                  | `pnpm test`                           |
| Test (single file)          | `npx vitest run tests/<file>.test.ts` |
| Lint                        | `pnpm lint`                           |
| Format (changed files only) | `pnpm prettier --write <path>`        |
| Smoke test                  | `pnpm smoke`                          |

**Package manager**: pnpm. **Node**: >=22. **Module system**: ESM (`"type": "module"`).

## Architecture

```
src/
├── index.ts              Entry point — wires server to stdio ACP stream
├── server.ts             ZcodeAcpServer — shared state + handler registration
├── backend/              ZCode subprocess client (JSON-RPC over stdio)
│   ├── client.ts         Spawns + communicates with zcode app-server
│   ├── credentials.ts    Reads ~/.zcode/v2/config.json for GLM API key
│   ├── listener.ts       EventStreamListener — subscribes to session/events
│   └── types.ts          ZCode protocol types
├── handlers/             ACP method handlers
│   ├── session.ts        session/new, session/prompt (turn loop), load, resume
│   ├── slash.ts          Slash-command interception (/compact, /mcp, etc.)
│   ├── account.ts        account/usage_stats — plan quota for remote clients
│   ├── extensions.ts     ZCode extensions (fork, compact, goal, model, mode, …)
│   ├── dispatch.ts       InternalEvent → ACP session/update dispatch
│   ├── replay.ts         Tail replay: load limit, load_earlier pagination
│   ├── io.ts             Client notification helpers
│   └── server-requests.ts  Server→client requests (permission, elicitation)
├── config/               Discovery + runtime config
│   ├── plugin-commands.ts  Load plugin commands from ~/.zcode/cli/
│   ├── skill-discovery.ts  Discover Skills from filesystem
│   ├── mcp-discovery.ts    Discover MCP servers from config + plugins
│   ├── auto-compact.ts     Threshold-based auto-compaction
│   ├── options.ts          Config options (model/mode/thought dropdowns)
│   └── runtime-model.ts    Model switching overlay
├── translators/          ZCode event → ACP translation
│   ├── event-translator.ts  Stream event → InternalEvent
│   ├── projection-differ.ts  Snapshot diff for turn-completion reconciliation
│   └── tool-helpers.ts       Diff builder, location extractor
├── interaction/          Permission, ExitPlanMode, AskUserQuestion handling
├── remote/               Remote access (opt-in via ZCODE_ACP_REMOTE=1)
│   ├── broadcast.ts      ClientRegistry + broadcast proxy (notify fan-out, request first-wins)
│   ├── config.ts         ENV parsing (gate, mandatory token, hub/bridge ports)
│   ├── endpoint.ts       Loopback ACP endpoint + hub registration heartbeat
│   ├── file-endpoint.ts  Read-only /fs/list + /fs/file, session-root scoped (ADR-0004)
│   └── hub-server.ts     Hub daemon: auth, discovery, byte-level proxy (ACP WS + /fs files), ?probe=1 liveness
├── quota/                GLM Coding Plan usage API client (/quota command)
├── cli.ts                Unified CLI entry (`zcode-acp`): subcommand dispatch
│                         (bare invocation → REPL) (ADR-0007)
├── repl/                 Interactive REPL (bare `zcode-acp`): Ink UI + ACP client
│   ├── model.ts          Pure turn state machine + idle status fold (commands,
│   │                     model/mode/thought selects, completion candidates,
│   │                     editor wrap/caret math, local /help & listings)
│   ├── App.tsx           Ink components — native-scrollback transcript
│   │                     (<Static>, Claude Code model) + compact dynamic
│   │                     footer (live-turn tail, queue panel, permission/
│   │                     question/session pickers, completion menu, wrapped
│   │                     input box). No alternate screen, no wheel capture.
│   ├── input-buffer.ts   Pure caret-editing line editor (code-point caret,
│   │                     Ctrl-B/F/A/E/U chords) — no React, testable
│   ├── history.ts        Per-project prompt history (JSONL under
│   │                     ~/.zcode/acp/repl-history), pure + testable
│   └── run.ts            Orchestration: spawn bridge, pump updates
└── bin/
    ├── hub.ts            Hub daemon entry (`zcode-acp hub`; spawned by absolute path)
    └── quota.ts          Quota cards entry (`zcode-acp quota`)
```

**Key boundary**: `backend/` talks to the ZCode subprocess. `handlers/` talks to
the ACP client (editor). `translators/` bridges the two event models. Never mix
ZCode protocol types into ACP notifications directly — always translate.

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [william0wang/zcode-acp](https://github.com/william0wang/zcode-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
