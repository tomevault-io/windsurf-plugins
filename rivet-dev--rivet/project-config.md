---
trigger: always_on
description: Design constraints, invariants, and reference commands for the Rivet monorepo. For implementation details, wiring, and procedural gotchas, follow the links under [Reference Docs](#reference-docs).
---

# CLAUDE.md

Design constraints, invariants, and reference commands for the Rivet monorepo. For implementation details, wiring, and procedural gotchas, follow the links under [Reference Docs](#reference-docs).

## Terminology

**Always spell the product name `agentOS`, never `AgentOS`; do not alter type identifiers such as `AgentOSActorConfig`.**

**ALWAYS use `rivet.dev` - NEVER use `rivet.gg`**

- API endpoint: `https://api.rivet.dev`
- Cloud API endpoint: `https://cloud-api.rivet.dev`
- Dashboard: `https://dashboard.rivet.dev`
- Documentation: `https://rivet.dev/docs`

**Use "sandbox mounting" when referring to the agentOS sandbox integration.** Do not use "sandbox extension" or "sandbox escalation." The feature mounts a sandbox as a filesystem inside the VM.

**ALWAYS use `github.com/rivet-dev/rivet` - NEVER use `rivet-dev/rivetkit` or `rivet-gg/*`**

**Never modify an existing published `*.bare` runner protocol version unless explicitly asked to do so.**

- Add a new versioned schema instead, then migrate `versioned.rs` and related compatibility code to bridge old versions forward.
- When bumping the protocol version, update `PROTOCOL_MK2_VERSION` in `engine/packages/runner-protocol/src/lib.rs` and `PROTOCOL_VERSION` in `rivetkit-typescript/packages/engine-runner/src/mod.ts` together. Both must match the latest schema version.

**Always use versioned BARE (`vbare`) instead of raw `serde_bare` for any persisted or wire-format encoding unless explicitly told otherwise.** Raw `serde_bare::to_vec` / `from_slice` has no version header, so any future schema change forces hand-rolled `LegacyXxx` fallback structs. `vbare::OwnedVersionedData` plus a versioned `*.bare` schema is the standard pattern. Acceptable raw-bare exceptions: ephemeral in-memory encodings that never cross a process boundary or hit disk, and wire formats whose protocol version is coordinated out-of-band (e.g. an HTTP path like `/v{PROTOCOL_VERSION}/...` or another channel that pins both peers to one schema per call).

- Avoid raw `f64` fields in vbare protocol schemas that use hashable maps; generated Rust derives `Eq`/`Hash`, so encode floats as fixed bytes or an ordered wrapper.
- Version converters must manually map fields between versions; never use serialize-deserialize round trips such as `transcode_version` or `serde_bare::to_vec` plus `from_slice`.
- RivetKit client/server protocol compatibility assumes the server/runtime is newer than the client; clients send their latest request protocol version, and servers handle older-client compatibility and negotiation.

When talking about "Rivet Actors" make sure to capitalize "Rivet Actor" as a proper noun and lowercase "actor" as a generic noun.

**Actor Runtime Socket** is the product name for the generic actor-local protocol. SQLite is its first capability and Unix sockets are its current transport; do not use "Actor SQLite UDS API" or "SQLite UDS" as the product name.

## Commands

### Build + test

```bash
# Check a specific package without producing artifacts (preferred for verification)
cargo check -p package-name

# Build
cargo build
cargo build -p package-name
cargo build --release

# Test
cargo test
cargo test -p package-name
cargo test test_name
cargo test -- --nocapture
```

### Development

```bash
# Run linter (but see "Development warnings" below)
./scripts/cargo/fix.sh

# Check for linting issues
cargo clippy -- -W warnings
```

- Agents may run `node scripts/format/agent-format.mjs` to format changed Biome and Rust files. Do not run broad `cargo fmt` or `cargo fmt --all` manually.
- Do not run `./scripts/cargo/fix.sh` or other broad formatter/fixer commands yourself.
- Ensure lefthook is installed and enabled for git hooks (`lefthook install`).

### Docker dev environment

```bash
cd self-host/compose/dev
docker-compose up -d
```

- Do not edit `self-host/compose/dev*` configs directly. Edit the template in `self-host/compose/template/` and rerun (`cd self-host/compose/template && pnpm start`) to regenerate.
- Rebuild publish base images with `scripts/docker-builder-base/build-push.sh <base-name|all> --push`. Update `BASE_TAG` when rebuilding shared builder bases; engine bases are published per commit in `publish.yaml`.

### Version control (jj)

- This repo uses jj (Jujutsu) on top of git. **jj's workflow is inverted from git:** the working copy is itself a revision that auto-tracks edits, so you create a new revision *before* making changes (with `jj new`) rather than committing *after* (`git commit`). The description is set separately via `jj describe`. There is no staging step.
- Before making changes, check whether jj is initialized by running `jj status`. If it fails (e.g. "There is no jj repo in '.'"), run `jj git init --colocate` from the repo root so jj lives alongside the existing `.git` directory. Do NOT run `jj git init` without `--colocate` — that creates a standalone jj repo and breaks the git workflow.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rivet-dev/rivet](https://github.com/rivet-dev/rivet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
