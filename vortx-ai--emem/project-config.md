---
trigger: always_on
description: This file is the README for *coding agents* working on the emem source.
---

# AGENTS.md

This file is the README for *coding agents* working on the emem source.
Per the [agents.md convention](https://agents.md) (OpenAI Codex, 2025;
Linux Foundation AAIF, Dec 2025): build commands, test rules, code style,
and what's off-limits during autonomous runs. End users of the protocol
should read [docs/agents.md](docs/agents.md) and [skills.md](web/skills.md)
instead; those describe how to *use* emem from an agent.

## First 5 minutes (using emem as an agent)

If you only want to *call* emem (not edit its source), here is zero to a
signed answer in three calls:

1. **Ask.** `POST /v1/ask {"question":"what is the NDVI near Mount Fuji?"}`
   (MCP tool `emem_ask`, or `emem_intent` / `POST /v1/intent` for a
   structured single-shot). The classifier picks the primitive and
   returns a signed receipt. This is the fastest path.
2. **Or take control.** `POST /v1/locate {"place":"Mount Fuji"}` → a
   `cell64`, then `POST /v1/recall {"cell":"<cell64>"}` (auto-materialises
   on a miss).
3. **Cite it.** Verify the receipt offline at `/verify` or via
   `POST /v1/verify_receipt`; hand other agents an `emem:fact:` token from
   `emem_memory_token` (or `emem_memory_bundle` for several).

Newer "connect & evolve" surfaces (typed temporal edges, the
contradiction-fed refinement loop) are walked end-to-end in
[examples/connect-and-evolve.md](examples/connect-and-evolve.md). Full
usage guide: [docs/agents.md](docs/agents.md). The rest of this file is
for agents editing the source.

## Repo shape

Rust workspace, 19 crates, version 2.4.0, MSRV 1.91. `emem-guard` is the verdict server for AI inference checkpoints (Anthropic Inference hooks, Claude Code hooks); it is a separate binary and shares no code path with the responder. The bulk of the code
lives in `crates/emem-api-rest/src/lib.rs` (HTTP/MCP router plus every inline materializer) and
`crates/emem-fetch/src/*.rs` (27 data connectors + 7 utility modules).
Earlier versions ran Clay, Prithvi, Galileo and JEPA-v2 on a GPU sidecar;
facts they signed still verify. Web surface in `web/` is plain HTML, no build step, included via
`include_str!`.

## Build

```sh
# Full workspace, debug
cargo build --workspace

# Production binary (what /home/ubuntu/.config/systemd/user/emem-server.service runs)
cargo build --release --bin emem-server

# After release rebuild on systems that need port 443 binding:
sudo setcap 'cap_net_bind_service=+ep' target/release/emem-server
systemctl --user restart emem-server.service
```

`scripts/redeploy.sh` does build + setcap + restart in one shot.

## Test

```sh
# Unit + bin + crate-level integration, no network
cargo test --workspace --lib --bins --tests

# Network-gated tests live in crates/emem-fetch/tests/live_cog_fetch.rs
# and skip automatically when offline. There is NO `live` cargo feature.
# Don't write `--features live`, that flag doesn't exist.
cargo test --workspace --test live_cog_fetch
```

Unit tests live inline in `src/` next to the code they cover (`#[cfg(test)] mod tests`).
Crate-level integration tests live under `crates/<crate>/tests/`. Only two
crates have those today: `emem-fact` (round-trip) and `emem-fetch` (live COG fetch).

## Lint + format

```sh
cargo fmt --all                                                  # local
cargo fmt --all --check                                          # CI gate
cargo clippy --workspace --all-targets -- -D warnings            # CI gate
```

Both gates run in `.github/workflows/ci.yml` against Linux + macOS, plus
an MSRV 1.91 build job. Don't ship a commit that fails either: the CI
will reject it and the next agent will have to figure out why.

Two hard-won specifics. CI pins `dtolnay/rust-toolchain@stable`, so the
clippy gate tracks CURRENT stable, not the MSRV: run `rustup update
stable` first (never while a build is running; it deletes the old
toolchain's rlibs out from under an in-flight compile) or new-in-stable
lints will pass locally and fail in CI one at a time. And run clippy
with `--all-targets`: a plain `--lib` test run never compiles the
integration-test and demo-bin targets, which is exactly where CI finds
what you missed.

## Code style

- **No `unsafe`.** Every crate carries `#![forbid(unsafe_code)]` at the top.
- **No `unwrap()` on user-facing paths.** Use `?` with structured errors;
  the `MaterializeMiss` / `CidNotFound` types are designed to surface
  honest gaps without panicking. `unwrap()` is fine inside tests.
- **Don't widen `pub`.** Keep crate boundaries tight. New public API
  needs a justification in the commit message.
- **Comments**: only when the *why* is non-obvious (a hidden constraint, a
  workaround for a specific bug, a subtle invariant). Don't explain *what*
  the code does; well-named identifiers do that. Don't reference the
  current task or PR; those belong in the PR description.
- **Receipts and CIDs are load-bearing**. Any change to
  `crates/emem-storage/src/server.rs::sign_receipt` or the canonical-CBOR
  layout in `crates/emem-fact/src/` requires a corresponding update to
  `docs/protocol.md` (the byte-by-byte preimage example) and to the
  in-browser verifier at `web/humans.html` (the BLAKE3 + Ed25519 path).
  These three places must always agree.

## Commit + PR conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vortx-AI/emem](https://github.com/Vortx-AI/emem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
