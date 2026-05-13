---
trigger: always_on
description: SPDX-FileCopyrightText: © 2025 StreamKit Contributors
---

<!--
SPDX-FileCopyrightText: © 2025 StreamKit Contributors

SPDX-License-Identifier: MPL-2.0
-->

# StreamKit Agent Notes

These notes apply to coding agents (Claude/Codex/Devin/etc.) contributing to
this repo. Agent-assisted contributions are welcome, but should be **supervised**
and **reviewed by a human** before merge.

## What is StreamKit

StreamKit is a self-hostable media processing server (Rust). A single binary
(`skit`) runs pipelines as a node graph (DAG) — via a web UI, YAML, or
WebSocket API. Two modes: **dynamic** (real-time, hot-reconfigurable) and
**oneshot** (stateless batch processing). See `agent_docs/architecture.md` for
the full architecture.

## Codebase Map

| Directory | Purpose |
|-----------|---------|
| `apps/skit/` | Server binary — HTTP/WS handlers, config, auth, plugins |
| `apps/skit-cli/` | CLI client binary (`skit-cli`) |
| `crates/core/` | Shared traits/types — `ProcessorNode`, `Pin`, `Packet`, `NodeRegistry` |
| `crates/engine/` | Pipeline executor — graph builder, oneshot engine, dynamic actor |
| `crates/nodes/` | Built-in nodes: `audio::`, `video::`, `transport::`, `core::`, `containers::` |
| `crates/api/` | YAML pipeline parsing, WebSocket protocol, TS type generation |
| `crates/plugin-{native,wasm}/` | Plugin host adapters (FFI and WASM) |
| `sdks/plugin-sdk/` | Plugin SDKs for Rust, Go, and C |
| `ui/` | React 19 web UI (Vite + Bun) |
| `plugins/native/` | Official ML plugins (Whisper, Kokoro, NLLB, etc.) |
| `samples/` | Example pipelines (`dynamic/` and `oneshot/`), audio files, images, fonts, Slint files |
| `tests/` | Pipeline validation tests (oneshot pipeline smoke tests) |
| `e2e/` | Playwright end-to-end tests |
| `docs/` | Astro + Starlight docs site (sidebar in `docs/astro.config.mjs`) |

## Tech Stack

- **Rust** (version pinned in `rust-toolchain.toml`), tokio, axum, wgpu
- **UI:** React 19, TypeScript, Zustand, Jotai, React Query, Radix UI, React Flow
- **Build/tooling:** `just` (task runner), Bun (UI), sccache (Rust build cache)
- **Testing:** `cargo test` (Rust), Vitest (UI), Playwright (E2E)
- **Platform:** Linux x86_64

## Workflow

- Keep PRs focused and minimal.
- Run `just test` and `just lint` before submitting (or explain why you couldn't).
- Follow `CONTRIBUTING.md`: DCO sign-off (`git commit -s`), Conventional
  Commits, SPDX license headers on all new files.
- **Linting discipline**: Do not blindly suppress lint warnings with
  ignore/exception rules. Refactor instead. If a suppression is truly necessary,
  include a comment explaining the rationale.
- **UI tooling**: Use `bun install` / `bunx` / `bun run` — never npm or pnpm.

## Comment Guidelines

> *"NEVER try to explain HOW your code works in a comment … just tell
> people WHY."*
> — [Linux kernel coding style, §8](https://www.kernel.org/doc/html/latest/process/coding-style.html)

> *"The best code is self-documenting. Giving sensible names to types and
> variables is much better than using obscure names that you must then
> explain through comments."*
> — [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Comments)

Default is **no comment**. If you feel compelled to add one, first ask
whether a better name or a small extraction would make it unnecessary.
Comments that restate the *how* (what the code obviously does) add noise
and drift out of sync; comments that explain the *why* (constraints,
trade-offs, gotchas invisible from the code alone) prevent real bugs.

### Do NOT write

These antipatterns accounted for ~3,500 lines removed in the v0.5 cleanup.

- **Line narration** — restating what the next line does
  (`// Send packet` before `send_packet()`, `// Check if empty` before
  `if x.is_empty()`). The [Google C++ guide](https://google.github.io/styleguide/cppguide.html#Implementation_Comments)
  limits implementation comments to "tricky, non-obvious, interesting, or
  important parts"; obvious code needs nothing.
- **`// Helper: X`** labels on descriptively-named functions — the name
  already tells the reader; a label adds nothing.
- **Verbose JSDoc / `///` on self-documenting items** — an essay on
  `PARAM_THROTTLE_MS = 33` or per-field docs like
  `/** Handler for slider onChange event */` adds zero information.
  [Rust RFC 505](https://rust-lang.github.io/rfcs/0505-api-comment-conventions.html)
  and the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/documentation.html)
  recommend a **single-line summary** as the first doc comment line;
  anything beyond that should earn its keep.
- **Section dividers** — `// --- Public Modules ---`, `// State`,
  `// Handlers`. Use blank lines or code structure instead.
- **Step-by-step numbered narration** — `// 1. Validate`, `// 2. Connect`.
  Extract named functions instead of numbering prose.
- **Complexity apologies** — multi-paragraph justifications above lint
  suppressions. Keep the rationale to one line.
- **Standard framework behavior** — `"useState setters are stable"`,
  `"useMemo prevents re-renders"`. Any React/Tokio/Axum developer knows this.
- **Dead code "for reference"** — git history preserves it. Delete it.
  As Google's [documentation best practices](https://google.github.io/styleguide/docguide/best_practices.html)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [streamer45/streamkit](https://github.com/streamer45/streamkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
