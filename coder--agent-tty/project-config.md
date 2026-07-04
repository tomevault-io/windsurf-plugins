---
trigger: always_on
description: You are an experienced, pragmatic software engineering AI agent. Do not over-engineer a solution when a simple one is possible. Keep edits minimal.
---

You are an experienced, pragmatic software engineering AI agent. Do not over-engineer a solution when a simple one is possible. Keep edits minimal.

Follow these instructions unless they conflict with higher-priority instructions or make the task impossible. Treat destructive actions, running-session deletion, public API/JSON contract changes, and public skill behavior as hard-stop areas: ask before making an exception. For ordinary judgment calls, choose the smallest safe path, state the assumption, and verify it.

# Operating Contract

Goal: complete the user's repo task end to end with the smallest safe change.

Success means:

- Relevant code, tests, and docs were inspected before edits.
- Every changed line traces directly to the request.
- Public CLI JSON, protocol schemas, event logs, and artifacts remain consistent.
- Assertions or Zod validation guard non-obvious assumptions.
- Targeted validation was run, or the reason it could not run is stated.
- The final response names what changed and the exact checks performed.

Stop and ask only when the missing information would materially change the implementation, cause irreversible side effects, or require overriding a hard project invariant.

# Project Overview

`agent-tty` is a CLI-first terminal automation tool for AI agents and humans. It creates long-lived PTY-backed sessions, exposes machine-friendly commands to control them, and produces inspectable artifacts such as semantic snapshots, PNG screenshots, asciicast recordings, and WebM exports.

The current implementation is a TypeScript/Node v1 with these main building blocks:

- **Commander** for the CLI surface (`src/cli/main.ts`).
- **node-pty** for PTY/process lifecycle.
- **Zod** for protocol, manifest, and artifact validation.
- **ghostty-web + Playwright** as the reference renderer for screenshot, wait, snapshot, and replay/export flows.
- **Vitest, Oxlint, Oxfmt, and TypeScript** for quality gates.
- **mise** as the canonical task runner in CI.

Session state is stored under `~/.agent-tty` by default. In tests and automation, prefer an isolated absolute `AGENT_TTY_HOME` instead of writing into the real home directory.

# Reference

## Important files

- `src/cli/main.ts` — public CLI contract and command registration.
- `src/cli/commands/*.ts` — command implementations; most behavior changes start here.
- `src/host/hostMain.ts` — per-session host orchestration for PTY, renderer, RPC, waits, and artifacts.
- `src/host/eventLog.ts` — append-only `events.jsonl` writer; append-time sequence numbers must stay contiguous.
- `src/host/replay.ts` — validated replay-input builder for manifest, dimensions, and target sequence semantics.
- `src/protocol/schemas.ts` and `src/protocol/messages.ts` — machine-facing schemas and result shapes.
- `src/storage/` — path guards, home/session resolution, manifest I/O, artifact manifests, and the persisted event-log codec.
- `src/renderer/ghosttyWeb/backend.ts` — reference renderer and Playwright browser harness.
- `src/export/asciicast.ts` and `src/export/webm.ts` — recording export logic.
- `src/util/assert.ts` — shared fail-fast assertion helpers.
- `design/ARCHITECTURE.md` — stable architecture and product intent overview.
- `RELEASE.md` — supported scope at the repo root.
- `dogfood/README.md` and `dogfood/CATALOG.md` — proof-bundle navigation and reviewer-facing validation artifacts.

## Important directories

- `src/cli/` — CLI entrypoint, output envelopes, and user-facing commands.
- `src/host/` — long-lived session host, event logging, replay, RPC.
- `src/renderer/` — renderer abstraction plus the `ghostty-web` reference backend.
- `src/storage/` — filesystem layout and manifest/artifact helpers.
- `src/protocol/` — Zod schemas, envelopes, and command/result types.
- `test/unit/` — focused unit tests with mocked dependencies.
- `test/integration/` — CLI-level behavior against isolated temp homes.
- `test/e2e/` — higher-level fixture-driven flows that assert rendered output and artifacts.
- `test/fixtures/apps/` — tiny terminal apps used by e2e and dogfooding.
- `design/` — architecture references and archived planning/status docs.
- `docs/` — contributor and maintainer workflow docs.

## Architecture

Treat the architecture as:

`CLI -> per-session host -> PTY + append-only event log -> renderer replay -> artifact manifests/files`

Important implications:

- The **CLI JSON envelope** is the stable automation surface.
- The **per-session host** is internal implementation detail.
- The **event log** is canonical execution truth.
- The **renderer** provides reference visual truth, not native-terminal parity.
- Artifacts should be reproducible from session state and replay data, not from ad hoc side channels.

# Essential commands

Preferred setup uses `mise`; fall back to direct `aube` only when necessary.

```sh
mise install
mise run bootstrap
```

If `mise` is unavailable but `aube` is available:

```sh
aube exec playwright install chromium
```

Core commands:

```sh
mise run build          # or: npm run build
mise run format         # or: npm run format
mise run format-check   # or: npm run format:check
mise run lint           # or: npm run lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/agent-tty](https://github.com/coder/agent-tty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
