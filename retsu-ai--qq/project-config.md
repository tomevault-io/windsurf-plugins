---
trigger: always_on
description: QQ is a local-first Rust toolkit for building, running, and orchestrating many AI
---

# AGENTS.md

## Project

QQ is a local-first Rust toolkit for building, running, and orchestrating many AI
agents. It ships as one `qq` binary for interactive terminal use, direct
automation, and a long-running HTTP/SSE server. Speed and developer experience
are product requirements; correctness, durability, and safe execution are
baseline constraints.

Read `docs/design/architecture.md` before changing system boundaries. It
records the initial direction, not a license to pre-build deferred features.

Documentation is organized in `docs/README.md`: `design/` for the system as
built, `adr/` for decisions, `plans/` for what is next, `plans/progress/` for
what is in flight, `runbooks/` for procedures. Before starting or reviewing a
unit of work, read `docs/plans/workflow.md`; it defines slices, ledgers,
review, and escalation. Record progress in the plan's ledger every session and
reserve ADR numbers in `docs/plans/progress/root.md`.

## Repository Map

- `src/`: binary, CLI, runtime composition, and authenticated model discovery.
- `crates/qq-auth/`: provider OAuth flows and credential storage.
- `crates/qq-client/`: HTTP/SSE client, reconnect/replay, client port, and the
  shared session state/reducer (`state`).
- `crates/qq-config/`: layered configuration, policy, and provider presets.
- `crates/qq-core/`: agent runtime, sessions, tools, and persistence behavior.
- `crates/qq-mcp/`: MCP client transport and tool discovery.
- `crates/qq-provider/`: provider-neutral model API and provider adapters.
- `crates/qq-protocol/`: shared commands, events, identifiers, and wire types.
- `crates/qq-reasoning/`: shared reasoning event vocabulary.
- `crates/qq-server/`: HTTP/SSE server and local-instance discovery.
- `crates/qq-tui/`: terminal UI and terminal-only client state.
- `xtask/`: repository automation; invoke it with `cargo xtask`. Releases are
  cut with `cargo xtask release X.Y.Z` in a PR, then `cargo xtask release
  --tag` on the merged `main` (see `docs/runbooks/release.md`).
- `website/`: the landing page and docs site (Astro + Starlight), generated
  from `docs/guide/` at build time and deployed to GitHub Pages; run with
  `nub` (see `docs/runbooks/website.md`). Edit the guide, not the site.

Keep dependencies pointed toward the narrow protocol and provider interfaces.
The root package is the composition root and translates external configuration
into crate-specific settings.

## Developer Workflow

The pinned stable Rust toolchain includes `rustfmt` and Clippy. A Nix development
shell is also available.

```sh
nix develop
cargo run -- ask "Reply with pong"
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test -p qq-provider --no-default-features --features test-support
```

The last line covers the minimal embedding profile (no Amazon Bedrock family,
no AWS SDK closure); run it when touching `qq-provider` manifests, features,
or anything under `aws.rs`, `providers/bedrock.rs`, or `providers/mantle.rs`.

Run the narrowest useful test while iterating, then run the workspace checks
before opening a PR. Run
`cargo bench -p qq-provider --bench provider_compiler` when changing provider
compilation or its hot path.

Never commit secrets, local credentials, `target/`, or generated build output.

## Engineering Priorities

1. Optimize for fast, responsive orchestration of many concurrent agents. No
   avoidable lag in startup, time to first token, streaming, tools, persistence,
   replay, or rendering.
2. Prefer the simplest design that is fast and easy to use and maintain. Do not
   trade developer speed for speculative flexibility or complex features.
3. Preserve correctness and durable state. Persist authoritative events before
   publishing them; make retries idempotent where work may be repeated.
4. Keep resource use predictable. Bound tasks, queues, channels, caches, output,
   and concurrency; apply backpressure and cancellation.
5. Measure meaningful hot paths. Support performance complexity with benchmarks
   and optimize end-to-end latency rather than isolated microbenchmarks.

Do not introduce placeholder crates, framework layers, generic extension points,
or alternate protocols without a concrete implemented need. Add dependencies
only for behavior being shipped.

## Rust Style

- Write safe, stable, idiomatic Rust and retain `#![forbid(unsafe_code)]`.
- Keep logic in one function or method unless an extracted unit is genuinely
  reusable, composable, or creates a meaningful interface. Do not create helpers
  merely to make a function shorter.
- Do not reach for `?` by default. Handle expected failures explicitly. When
  propagation is the correct behavior, preserve context and map the failure into
  a specific error type rather than erasing it.
- Prefer domain-specific error enums, normally derived with `thiserror`. Make
  error variants actionable and preserve sources. Avoid stringly typed errors
  and broad `Box<dyn Error>` in library interfaces.
- Use structs and enums to encode invariants and impossible states. Keep public
  interfaces small and choose ownership deliberately.
- Avoid unnecessary allocation, cloning, boxing, dynamic dispatch, and data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [retsu-AI/qq](https://github.com/retsu-AI/qq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
