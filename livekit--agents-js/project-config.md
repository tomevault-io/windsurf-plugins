---
trigger: always_on
description: SPDX-FileCopyrightText: 2026 LiveKit, Inc.
---

<!--
SPDX-FileCopyrightText: 2026 LiveKit, Inc.

SPDX-License-Identifier: Apache-2.0
-->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LiveKit Agents for Node.js — a TypeScript framework for building realtime, multimodal, and voice AI agents that run on servers. This is the Node.js distribution of the [LiveKit Agents framework](https://github.com/livekit/agents) (originally Python).

## Monorepo Structure

- **`agents/`** — Core framework (`@livekit/agents`). Agent orchestration, LLM/STT/TTS abstractions, voice pipeline, metrics, telemetry, IPC/process pooling, and the CLI.
- **`plugins/`** — Provider plugins (`@livekit/agents-plugin-*`). Each implements one or more of: LLM, STT, TTS, VAD, EOU (end-of-utterance), Realtime, or Avatar.
- **`examples/`** — Example agents (private, not published). Run with `pnpm build && node ./examples/src/<file>.ts dev`.
- **`tests/e2e/`** — End-to-end tests via Docker (separate from unit tests co-located in each package).

**Tooling:** pnpm 9.7.0 workspaces, Turborepo for builds, tsup for bundling (CJS + ESM), TypeScript 5.4+, Vitest for tests, Changesets for versioning.

## Common Commands

```bash
pnpm build                  # Build all packages (turbo)
pnpm build:agents           # Build only @livekit/agents and its deps
pnpm build:plugins          # Build only plugins and their deps
pnpm clean:build            # Clean dist/ dirs then rebuild
pnpm test                   # Run all tests (vitest)
pnpm test -- --testPathPattern=agents/src/llm  # Run tests by path
pnpm test -- --testNamePattern="chat context"  # Run tests by name
pnpm test:watch             # Watch mode
pnpm lint                   # ESLint all packages
pnpm lint:fix               # ESLint with auto-fix
pnpm format:check           # Prettier check
pnpm format:write           # Prettier format
pnpm api:check              # API Extractor validation
pnpm api:update             # Update API declarations
```

### Running an example agent

```bash
pnpm build && node ./examples/src/basic_agent.ts dev --log-level=debug
```

Required env vars: `LIVEKIT_URL`, `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`, plus provider keys (e.g. `OPENAI_API_KEY`).

> **Rule (must follow before handing off to the user to run any example/agent file):**
> Always run `pnpm build` first and iterate on TypeScript / dependency errors until it exits 0. Never tell the user to `node ./examples/src/<file>.ts` (or any `.ts` entrypoint) until the build succeeds — this includes after editing example files, after modifying framework code that breaks `tsc`, and after copy-pasting reproductions from external sources. If the build fails, fix the errors yourself (add missing deps, fix outdated API usage, update types) before asking the user to run anything.

### Pre-downloading plugin assets (for Docker builds)

`@livekit/agents` ships a `livekit-agents` bin with a `download-files` subcommand that discovers installed `@livekit/agents-plugin-*` packages from `node_modules` and runs each plugin's `downloadFiles()` (e.g. HuggingFace models for the turn detector) — without loading the user's agent code. Use it in Dockerfiles to separate dependency installation from app code so the download layer can be cached:

```dockerfile
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
RUN npx livekit-agents download-files
COPY src/ ./src
RUN pnpm build
```

The `download-files` subcommand on `cli.runApp()` is deprecated for this purpose since it requires loading the agent file first.

### Debugging individual plugins

Create a test file prefixed with `test_` in `examples/src/`. No `defineAgent` wrapper needed — just import the plugin directly and run:

```bash
pnpm build && node ./examples/src/test_my_plugin.ts
```

## Architecture

### Voice Pipeline (`agents/src/voice/`)

The core of the framework. Pipeline flow: **Audio In → VAD → STT → LLM → TTS → Audio Out**.

Key classes and their roles:

- **`Agent`** — Base class holding instructions, tools, and model config. Subclass to override pipeline hooks (`ttsNode`, `realtimeAudioOutputNode`, `sttNode`, `llmNode`). Use `voice.Agent.default.<hook>(this, ...)` to call the base implementation from overrides.
- **`AgentTask`** — Extends Agent for composable task workflows with isolated chat context and tool scoping.
- **`AgentSession`** — Orchestrates the full session lifecycle: connects to LiveKit room, manages turn detection, handles interruptions, collects metrics. Entry point: `session.start({ agent, room })`.
- **`AgentActivity`** (`agent_activity.ts`, ~100KB) — Complex state machine managing individual turns: VAD trigger → STT → endpointing → LLM inference → TTS generation → playout. Supports preemptive generation (starts LLM while user still speaking).
- **`SpeechHandle`** — Represents a unit of agent speech with lifecycle tracking and priority levels (`LOW=0`, `NORMAL=5`, `HIGH=10`). Sources: `'say'`, `'generate_reply'`, `'tool_response'`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [livekit/agents-js](https://github.com/livekit/agents-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
