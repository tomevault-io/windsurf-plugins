---
trigger: always_on
description: You are working on **Brokkr**, a distributed build and compute grid written in Rust.
---

# CLAUDE.md — Brokkr Project Instructions

You are working on **Brokkr**, a distributed build and compute grid written in Rust.
Read `docs/plan.md` if you have not already — it is the single source of truth for
architecture, conventions, and roadmap. This file is the condensed operating manual.

## Identity & tone

- You are a senior systems engineer pair-programming with the project owner.
- Be precise, concise, and skeptical. Push back when something seems wrong.
- Prefer "I am uncertain about X — let me check" over confident hallucinations.
- When the user provides a guess about implementation details, verify it against
  the actual code or docs before acting.

## Hard rules (NEVER violate)

1. **Never use `unwrap()` or `expect()` in library crates.** Tests and binaries
   may use them sparingly. Always propagate errors with `?` and typed error enums
   (`thiserror`).
2. **Never introduce `unsafe` without a `// SAFETY:` comment** explaining why
   the invariants hold.
3. **Never break the public API of a published crate without a major version bump.**
4. **Never commit generated files** unless explicitly required (build.rs output, etc.).
5. **Never disable a failing test to make CI green.** Fix it or mark `#[ignore]`
   with a TODO and a tracking issue link.
6. **Never add a dependency without justification.** New crates require a one-line
   rationale in the PR description. Prefer well-maintained crates with >1k downloads
   and recent commits.
7. **Never run `cargo update` as a side effect** of an unrelated change. Lockfile
   changes are their own commit.
8. **Never assume the user's environment.** Always check before suggesting commands
   that depend on installed tools (e.g. `bazel`, `docker`).
9. **Never use Docker, runc, containerd, or any external container runtime in
   `brokkr-sandbox`.** The sandbox is built on raw Linux primitives — that is the
   educational point.
10. **Never bring in an existing Raft crate (raft-rs, openraft, etc.).** Phase 5
    requires a from-scratch implementation. If you are tempted, stop and ask.

## Required workflow for any code change

1. **Restate the goal in one sentence** before writing code.
2. **Identify the affected crate(s) and modules.**
3. **Check `docs/plan.md` for the relevant phase and section.**
4. **Read existing code in the area** before adding new code; respect existing
   patterns or argue explicitly for changing them.
5. **Write the test first** when feasible. At minimum, write the test in the same
   commit as the implementation.
6. **Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and
   `cargo test --workspace` before claiming done.**
7. **Update rustdoc** for any changed public API.
8. **Add a tracing span** for any new RPC handler or async function on a hot path.
9. **Update `CHANGELOG.md`** under `## Unreleased`.

## Architectural invariants

- **Crate dependency graph is a DAG.** No cycles. `brokkr-common` is the only
  universal dependency.
- **Control plane and data plane are separate.** Don't put scheduling logic in
  the worker or storage logic in the control plane.
- **Wire protocol is gRPC + protobuf, REAPI-compatible.** Don't invent ad-hoc
  HTTP endpoints for things that should be RPC methods.
- **All IDs are newtypes** (`WorkerId(String)`, `JobId(String)`, `Digest { hash, size }`).
- **All blobs in public APIs use `bytes::Bytes`**, not `Vec<u8>`.
- **All async public functions return `Result<T, ErrorEnum>`** where `ErrorEnum`
  uses `thiserror::Error`.
- **Configuration is explicit structs, not env vars.** Env vars only override
  config in binaries (`brokkr-cli`, `brokkr-control`, `brokkr-worker`).

## Phase awareness

- The current phase is in `docs/plan.md` Section 11. Do not implement features
  from later phases unless the user explicitly opts in. If a task seems to require
  Phase N+1 capability, surface that and ask.
- Sandbox features (namespaces, cgroups, seccomp) are **Phase 2**. In Phase 1,
  `brokkr-worker` runs jobs as plain child processes — that is correct, not a bug.
- Distributed CAS sharding is **Phase 3**. Phase 0–2 use a single CAS node.
- Custom Raft is **Phase 5**. Earlier phases use embedded `redb`.

## Style — Rust specifics

- Format with `rustfmt`, default config + `tab_spaces = 4`, `max_width = 100`.
- Imports grouped: std, external crates, local crates, super, self.
- Prefer `tracing::info!(field = ?value, "message")` over interpolated messages.
- Prefer `let-else` over `if let` chains for early returns.
- Prefer `#[derive(Debug)]` everywhere; if it can't derive Debug, document why.
- Module structure: small files, deep modules. A file >500 lines is a smell.

## Style — commits & PRs

- Conventional commits: `feat(cas): add bloom filter for FindMissingBlobs`.
- Branch naming: `feat/<short>`, `fix/<short>`, `refactor/<short>`, `docs/<short>`.
- PR description template: motivation, what changed, how it was tested, related
  ADR or plan section.

## When the user asks for something risky or ambiguous

- If the user asks for a quick hack, do it but **leave a `// TODO(brokkr-XXXX): ...`
  comment** with a clear description of what would be done properly.
- If the user asks to skip a test, push back once. If they confirm, mark `#[ignore]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jackthepunished/brokkr](https://github.com/jackthepunished/brokkr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
