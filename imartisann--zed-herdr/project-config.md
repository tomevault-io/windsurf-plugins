---
trigger: always_on
description: `zed-herdr` is a private Bun/TypeScript application that keeps the active HerdR workspace available in an existing Zed session. HerdR 0.7.3 protocol 16 is authoritative for workspace state. The daemon consumes read-only HerdR snapshots/events and plugin cwd hints, resolves Git roots, then invokes only Zed's supported `zed -e <absolute-git-root>` command. It must not inspect Zed databases, replace windows, kill processes, or mutate existing HerdR panes.
---

# Repository Guidelines

## Project Overview

`zed-herdr` is a private Bun/TypeScript application that keeps the active HerdR workspace available in an existing Zed session. HerdR 0.7.3 protocol 16 is authoritative for workspace state. The daemon consumes read-only HerdR snapshots/events and plugin cwd hints, resolves Git roots, then invokes only Zed's supported `zed -e <absolute-git-root>` command. It must not inspect Zed databases, replace windows, kill processes, or mutate existing HerdR panes.

Supported hosts are macOS and Linux with Bun, Git, HerdR `>=0.7.3`, and the Zed CLI.

## Architecture & Data Flow

1. `index.ts` runs `runCli()` through `BunRuntime.runMain`; `src/cli.ts` accepts exactly one command: `daemon`, `hook`, `health`, or `toggle`.
2. `src/app.ts` composes Bun platform services, the HerdR workspace source, Zed adapter, JSON logger, plugin control server, and synchronization daemon through Effect `Layer`s.
3. `src/herdr/client.ts` bootstraps each generation with `session.snapshot`, opens `events.subscribe`, waits for `subscription_started`, and only then publishes an `Invalidated` event. A fresh snapshot is requested for synchronization. Disconnects cancel the generation and publish `Disconnected`.
4. `src/sync/daemon.ts` merges source events and hook cwd hints into queues. A single worker collapses bursts for 50 ms, gates every async stage against the live generation and runtime enable cycle, resolves all projects, atomically replaces cache state, ensures each unique Git root, and focuses the authoritative workspace.
5. `src/sync/resolve-project.ts` prefers HerdR `checkoutPath`, falls back to the matching hook cwd hint, verifies a directory, and runs `git -C <path> rev-parse --show-toplevel`. Invalid, inaccessible, ambiguous, or non-Git paths are logged and never reach the editor adapter.
6. `src/editor/zed.ts` serializes shell-free Zed calls. `ensureProject` caches only successful roots; `focusProject` always runs `zed -e`.
7. Plugin hooks send cwd hints to the owner-only control socket. When no live daemon exists, `src/plugin/hook.ts` coordinates contenders with a token lock and opens one unfocused HerdR daemon tab. `health` reads daemon identity; `toggle` pauses or resumes the live synchronization daemon; neither starts one.

Preserve these boundaries: transport types do not enter core domain types, stale generations cannot mutate cache or call Zed, and plugin control/hook code remains separate from the Effect-based synchronization core.

## Key Directories

| Path            | Purpose                                                                                        |
| --------------- | ---------------------------------------------------------------------------------------------- |
| `src/domain/`   | Effect `Schema` domain values and tagged error types independent of HerdR/Zed.                 |
| `src/services/` | `Context.Tag` contracts for workspace source, cwd hints, and editor adapter.                   |
| `src/sync/`     | Project resolution, generation-aware cache, debounce, serialization, and editor orchestration. |
| `src/herdr/`    | Protocol-16 schemas, NDJSON framing, Unix-socket client, and core source projection.           |
| `src/editor/`   | Timeout-safe, state-preserving Zed CLI adapter.                                                |
| `src/plugin/`   | Local control protocol/socket plus hook decoding, locking, and pane startup.                   |
| `test/`         | Bun unit, integration, socket-safety, and built-artifact E2E suites by subsystem.              |
| `dist/`         | Generated Bun bundle; ignored by Git and recreated by `bun run build`.                         |

There is no `src/index.ts`, scripts directory, or separate test configuration. Root `index.ts` and `package.json` are authoritative.

## Development Commands

Run commands from the repository root with Bun:

```bash
bun install --frozen-lockfile  # install exactly from bun.lock
bun run dev                    # watch index.ts and run the daemon
bun run start                  # run the source daemon
bun run build                  # bundle index.ts to dist/index.js for Bun
bun ./dist/index.js daemon     # run the built daemon
bun ./dist/index.js health     # query an existing matching daemon; does not start one
bun test                       # run all tests
bun test test/sync/daemon.test.ts  # run one suite directly
bun run typecheck              # tsc --noEmit
bun run lint                   # oxlint .
bun run lint:fix               # oxlint . --fix
bun run format                 # oxfmt .
bun run check                  # typecheck, lint, build, then full tests
```

Documented plugin workflow:

```bash
bun run build
herdr plugin link <repo>
herdr plugin enable artisann.zed-herdr
herdr plugin disable artisann.zed-herdr
herdr plugin unlink artisann.zed-herdr
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ImArtisann/zed-herdr](https://github.com/ImArtisann/zed-herdr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
