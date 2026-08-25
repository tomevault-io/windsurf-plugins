---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## Project overview

Offline Protocol SDK: an offline-first messaging protocol in Rust with
multi-transport switching (BLE, Wi-Fi Direct, Reticulum, Nostr, internet relay),
mesh networking, and automatic MLS end-to-end encryption (RFC 9420). Exposed to
iOS, Android, React Native and Python via UniFFI bindings.

## Where the knowledge lives

This file is repository instructions. The design knowledge lives in documents
that are versioned, reviewable, and readable by people who are not an agent.
**Read the relevant one before changing behaviour in its area.**

| If you are touching | Read first |
|---------------------|------------|
| Wire encoding, envelopes, control frames, negotiation | [docs/spec/](docs/spec/README.md) |
| Anything security-relevant | [docs/security/threat-model.md](docs/security/threat-model.md) |
| Acknowledgement, retry, session, group or transport behaviour | [docs/state-machines/](docs/state-machines/README.md) |
| A decision that looks odd or over-engineered | [docs/adr/](docs/adr/README.md) |
| `offline-protocol-core`: adding an import, a dependency, or a constructor | ADR [0020](docs/adr/0020-core-compiles-without-std.md) (it is dual std/no_std) |
| `offline-protocol-sealed`: the envelope codec, `derive_address`, canonical signing payloads, ratchet constants, the 1:1 control-frame prefixes, `KeyPackagePayload` | ADR [0022](docs/adr/0022-one-sealed-layer-shared-with-the-leaf.md) (also dual std/no_std, and the one home for each) |
| `offline-protocol-leaf`: anything a device does at pairing, on a frame, or with its store | ADR [0021](docs/adr/0021-a-leaf-node-speaks-mls.md) and [docs/spec/leaf-provisioning.md](docs/spec/leaf-provisioning.md) (a time source, real entropy, durable-before-emit and authorization are obligations, not suggestions) |
| Replicated documents: the store, sync frames, attachments | [docs/spec/data-sync.md](docs/spec/data-sync.md), [the replication state machine](docs/state-machines/data-replication.md), ADR [0018](docs/adr/0018-data-layer-engine-and-storage-seams.md) and [0019](docs/adr/0019-remote-document-imports-are-contained-not-trusted.md) |
| Any binding: Swift, Kotlin, Python, TypeScript | [docs/bridges/](docs/bridges/README.md) |

The ADR index is the fastest route to "why is this like this". If you are about
to simplify something that looks redundant, check there first: several shapes in
this codebase are correct only as a whole, and the ADR names the failure that
partial versions cause.

## Common commands

```bash
# Verify loop (lint subsumes typecheck; don't run a separate `cargo build` first,
# it only adds a third artifact set including the expensive uniffi cdylib link)
cargo clippy --workspace -- -D warnings
cargo test --workspace --lib                    # all unit tests; skips empty per-crate doctest passes

# Test
cargo test --workspace                          # full run incl. doctests (what CI runs)
cargo test --package offline-protocol-core      # single crate
cargo test test_message_creation                # single test
cargo test -- --nocapture                       # with stdout

# Build (only when you need compiled artifacts, e.g. the uniffi cdylib)
cargo build --workspace
cargo build --workspace --release

# Format (must pass before commits; fmt takes --all, not --workspace)
cargo fmt --all
cargo fmt --all -- --check

# Docs (CI gates this under -D warnings; without the flag broken intra-doc links
# only print and still exit 0, so the plain command passes what CI fails)
RUSTDOCFLAGS="-D warnings" cargo doc --workspace --no-deps

# Benchmarks (Criterion)
cargo bench --package offline-protocol-bench

# Bare metal. `offline-protocol-core`, `offline-protocol-sealed` and
# `offline-protocol-leaf` are dual std/no_std and CI gates all three no_std
# halves; nothing else in the workspace compiles without `std`, so a stray
# `use std::` in one of them only fails here. So does an mls-rs error
# formatted with `{}`: mls-rs implements Display only under std.
rustup target add thumbv8m.main-none-eabihf
for crate in offline-protocol-core offline-protocol-sealed; do
    cargo clippy -p "$crate" --no-default-features \
        --target thumbv8m.main-none-eabihf -- -D warnings
done
# The leaf crate needs a getrandom backend selected; the firmware registers one.
cargo clippy -p offline-protocol-leaf --no-default-features \
    --features bare-metal-rng --target thumbv8m.main-none-eabihf -- -D warnings
./tools/embedded-footprint/measure.sh    # flash/RAM cost of the protocol layer
```

### UniFFI and mobile builds

```bash
# Regenerate EVERY binding after a UDL change: Swift, Kotlin and Python together.
# The three are one artifact set from one UDL, so there is one script. A partial
# regeneration fails at runtime, not at build time.
./scripts/generate-bindings.sh

cd bindings/react-native
npm run build:uniffi:all          # all platforms
npm run build:uniffi:ios          # iOS only
npm run build:uniffi:android      # Android only
npm run generate:bindings         # wrapper for ../../scripts/generate-bindings.sh
```

Prerequisites: `cargo install uniffi --version 0.30.0 --features cli --locked`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Offline-Protocol/offline-protocol-sdk](https://github.com/Offline-Protocol/offline-protocol-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
