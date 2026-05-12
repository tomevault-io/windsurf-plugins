---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
npm run dev                       # run the CLI from source (./src/cli.js)
npm test                          # node --test, runs all tests in test/
node --test test/server-startup.test.js   # run a single test file
node --test --test-name-pattern="warmup" test/whiteboard-session.test.js  # filter by test name
npm run build:moonshine-sidecars  # build Python -> single-binary sidecars for macOS arm64+x64
node ./scripts/build-moonshine-sidecars.js darwin-arm64   # build only one target
```

There is no separate lint step. CI (`.github/workflows/ci.yml`) runs `npm ci && npm test` on Node 24.

The `--no-open` flag suppresses auto-launching the browser, which is useful when iterating from a terminal.

## Architecture

The product is a single Node process that serves a static Excalidraw frontend, runs an STT pipeline, and orchestrates an LLM agent that edits the whiteboard via tool calls. The end-to-end loop is:

```
browser mic -> WS audio frames -> transcription provider -> turn queue ->
runWhiteboardAgent (in src/server.js) -> tool call ops -> apply to scene ->
broadcast whiteboard:update over WS -> frontend re-renders Excalidraw
```

### Entry points and wiring

- `src/cli.js` parses args, loads `~/.config/autopreso/settings.json` via `settings-store.js`, resolves an agent provider, then calls `startServer`.
- `src/server.js` is the central hub. It owns the Express + WebSocket server, mounts the static frontend in `public/`, instantiates a `WhiteboardSession`, builds a `TranscriptionManager`, and exposes `runWhiteboardAgent` / `runWhiteboardWarmupOnce` which contain the system prompt and the AI SDK `tool({...})` definitions. `server.js` is large (~1000 LOC) on purpose - keep the agent prompt, message construction, and tool schemas colocated.
- `public/app.js` is the React frontend. It renders Excalidraw, handles mic capture at 24 kHz, sends audio frames over WS, periodically pushes downscaled screenshots back to the server (`whiteboard:screenshot`), and reflects server-pushed scene updates back into Excalidraw. Frontend is plain ES modules loaded via `<script type="importmap">` from esm.sh - no build step.

### Two-mode session model (`src/whiteboard-session.js`)

The session has two modes that are NOT symmetric:

- **`staging`** - client-side scratchpad. The server does not track elements in this mode; the frontend owns them. Used to seed the canvas with reference content before going live.
- **`live`** - the server owns `state.elements` as the source of truth. Audio, screenshots, and user edits all flow into the server, which applies agent edits and broadcasts updates.

Transitions: `POST /api/preso/start` builds a "staging primer" message (current scene snapshot + downscaled screenshot when staging is non-empty) and kicks off the warmup loop. `POST /api/preso/back-to-staging` returns to client-owned mode.

### Warmup loop

Before the user speaks, `startWarmupLoop` repeatedly fires the agent against the staging primer with exponential backoff (`DEFAULT_WARMUP_DELAYS`, max 8 attempts). Its purpose is **prompt cache priming**: after the loop ends, `agentHistory` is forced to `[warmup_user_msg, assistant("UNDERSTOOD")]` so every subsequent turn reuses the same prefix bytes. Do not change this primer-then-fixed-history pattern without understanding the cache implications.

### Transcript turn queue (`src/transcript-turn-queue.js`)

Transcript chunks are debounced (default 150 ms) and gated by an `isReady` predicate. While a turn is running, additional chunks are buffered and concatenated for the next turn. This means the agent never has more than one in-flight turn, but it always sees the most recent burst of speech in one shot. `isTrivialTranscript` in `whiteboard-session.js` filters out filler-only chunks ("uh", "okay", etc.) so they don't trigger turns on their own.

### Whiteboard edit model (`src/whiteboard-tools.js`)

The agent does not see Excalidraw JSON directly; it sees a **line-numbered text view** of the scene (`formatLineNumberedWhiteboard`) and emits `replace`, `insert_after`, or `delete` operations against line numbers. `applyWhiteboardEditOperations` validates and applies them in order. When changing the agent's contract, update both the tool schema in `server.js` and this applier, and add a test in `test/whiteboard-tools.test.js`.

### Agent providers (`src/agent-provider.js`, `src/codex-auth.js`)

Three providers, all routed through the `@ai-sdk/openai` adapter:

- **openai** - direct API key.
- **codex** - reads the user's Codex CLI auth from `~/.codex/auth.json`, then talks to the ChatGPT backend with that bearer token. No API key needed.
- **ollama** - OpenAI-compatible local endpoint (`http://localhost:11434/v1`).

`reasoningEffort` is validated against the set `{none, low, medium, high, xhigh}`.

### Transcription providers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/autopreso](https://github.com/kunchenguid/autopreso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
