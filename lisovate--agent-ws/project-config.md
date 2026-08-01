---
trigger: always_on
description: You are working on agent-ws, a standalone WebSocket bridge for CLI AI agents (Claude Code, Codex).
---

# agent-ws

You are working on agent-ws, a standalone WebSocket bridge for CLI AI agents (Claude Code, Codex).

## Project Overview

agent-ws is a TypeScript Node.js process that bridges any WebSocket client with CLI AI agents. It is a **dumb pipe** — no prompt engineering, no credential handling, just transport. Any client can connect: browser frontends, backend services, scripts, other CLI tools.

**Key principle: Local-first.** All AI processing happens on the user's machine using their existing CLI agent authentication.

## Tech Stack

- Node.js 20+ (TypeScript, ESM)
- ws (WebSocket server)
- commander (CLI argument parsing)
- pino (structured logging)
- esbuild (bundling)
- vitest (testing)

## How It Works

```
┌───────────────┐     WebSocket      ┌─────────────┐      stdio       ┌─────────────┐
│  Your App     │ <=================> │  agent-ws   │ <===============> │ Claude Code │
│  (any client) │   localhost:9999   │  (Node.js)  │      stdio       │  / Codex    │
└───────────────┘                    └─────────────┘                   └─────────────┘
```

1. User runs `agent-ws` on their machine
2. Agent starts WebSocket server on localhost:9999
3. Each connection gets its own CLI process
4. Client sends prompt → agent spawns the appropriate CLI agent
5. Response streams back in real-time

## Commands

```bash
npm install          # Install dependencies
npm run build        # Build TypeScript → dist/
npm test             # Run vitest tests
npm run typecheck    # TypeScript check
npm start            # Run from dist/
npm run dev          # Run with --watch
```

## Documentation

- When creating new docs for this repo, prefer standalone HTML/CSS by default: use semantic HTML, responsive accessible CSS, a readable max-width and line-height, clear visual hierarchy, and concise plain language.

## CLI Options

```
-p, --port <port>            WebSocket port (default: 9999)
-H, --host <host>            Hostname (default: localhost)
-m, --mode <mode>            Permission mode: safe, agentic, unrestricted (default: safe)
    --sandbox <preference>   Sandbox: auto, none, os, seatbelt, bwrap (default: none)
-c, --claude-path <path>     Path to Claude CLI (default: claude)
    --codex-path <path>      Path to Codex CLI (default: codex)
-t, --timeout <seconds>      Process timeout (default: 600)
    --log-level <level>      debug, info, warn, error (default: info)
    --origins <origins>      Comma-separated allowed origins
-V, --version                Show version
-h, --help                   Show help
```

## Project Structure

```
agent-ws/
├── src/
│   ├── index.ts               # Barrel export (library entry point)
│   ├── cli.ts                 # Commander CLI entry point
│   ├── agent.ts               # Orchestrator: wires server + logger
│   ├── server/
│   │   ├── websocket.ts       # WS server, heartbeat, per-connection state
│   │   └── protocol.ts        # Message types, validation
│   ├── process/
│   │   ├── base-runner.ts          # Abstract BaseRunner: spawn/kill/timeout, file-write sandbox, isWithin helper. Spawns through sandbox.wrapSpawn
│   │   ├── claude-runner.ts        # ClaudeRunner (extends BaseRunner): wires the parser, post-edit reads
│   │   ├── claude-stream-parser.ts # Stateless parser for Claude's stream-json NDJSON output
│   │   ├── codex-runner.ts         # CodexRunner (extends BaseRunner): JSONL parsing, thread resumption
│   │   ├── output-cleaner.ts       # ANSI stripping via node:util
│   │   └── sandbox/
│   │       ├── index.ts            # selectSandbox factory + probeAvailableSandboxes
│   │       ├── types.ts            # Sandbox interface + SandboxSpawnOpts/Result
│   │       ├── noop.ts             # NoopSandbox (default, no isolation)
│   │       ├── seatbelt.ts         # macOS sandbox-exec backend + buildSeatbeltProfile
│   │       └── bwrap.ts            # Linux bubblewrap backend + buildBwrapArgs
│   └── utils/
│       ├── logger.ts          # Pino logger factory
│       └── claude-check.ts    # CLI availability probe (used by capabilities)
├── test/
│   ├── protocol.test.ts       # Message parsing, validation
│   ├── base-runner.test.ts    # BaseRunner: caching, disposal, spawn failure
│   ├── claude-runner.test.ts  # Claude NDJSON stream parsing
│   ├── codex-runner.test.ts   # Codex JSONL stream parsing
│   ├── runner-args.test.ts    # CLI argument building for both runners
│   ├── output-cleaner.test.ts # ANSI/VT sequence stripping
│   ├── sandbox.test.ts        # Sandbox interface, Seatbelt/bwrap arg construction, BaseRunner integration
│   └── websocket.test.ts      # Integration: WS server, rate limiting, auth, capabilities handshake
├── build.js                   # esbuild bundler config
├── tsconfig.json
├── vitest.config.ts
└── package.json
```

## Message Protocol

Protocol version is `1.1` (`PROTOCOL_VERSION` in `src/server/protocol.ts`).

### Client → Agent

```typescript
{ type: "prompt", prompt: string, requestId: string, model?: string, provider?: "claude" | "codex", projectId?: string, systemPrompt?: string, thinkingTokens?: number, images?: PromptImage[], files?: PromptFile[] }
{ type: "cancel", requestId?: string }
{ type: "capabilities" }
```

- `requestId` max 256 chars.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lisovate/agent-ws](https://github.com/Lisovate/agent-ws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
