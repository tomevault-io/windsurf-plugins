---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

## What this is

Bulwark scans a Linux host for security misconfigurations and intrusion indicators using a native Rust rule engine over declarative YAML rules, and explains findings in plain language with a suggested fix. Design rationale, architecture, and alternatives-considered all live in `docs/guide/architecture.md` — read that before making an architectural change, not just this file. Background research grounding the rule checklist (Lynis, MITRE ATT&CK, HackTricks) is in `research/2026-07-11-linux-security-checklist/report.md`.

## Build & development commands

```bash
# Core + CLI
cargo build --workspace
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all                        # cargo fmt --all -- --check in CI
cargo run -p bulwarkctl -- scan
cargo run -p bulwarkctl -- rules validate rules/
cargo test -p bulwarkctl --test e2e -- --ignored --test-threads=1   # needs Docker; see below

# GUI (from apps/bulwark-app/)
npm install
cargo tauri dev                        # hot reload for frontend; Rust/tauri.conf.json changes need a restart
cargo tauri build                      # produces .deb, .rpm, .AppImage in target/release/bundle/
npx tsc --noEmit                       # type-check the frontend
npm run lint                           # eslint
npm run format:check                   # prettier --check

# Docs site (from docs/)
npm install
npm run dev                            # local preview
npm run build                          # static build to docs/.vitepress/dist

# Packaging (from workspace root, after a release build)
cargo build --release -p bulwarkctl
cargo deb -p bulwarkctl --no-build       # requires `cargo install cargo-deb`
cargo generate-rpm -p crates/bulwarkctl  # requires `cargo install cargo-generate-rpm`
```

CI (`.github/workflows/ci.yml`) runs fmt-check, clippy `-D warnings`, `cargo test --workspace`, `rules validate rules/`, and a frontend typecheck — run all of these locally before considering a change done.

### Releases

`.github/workflows/release.yml` builds and publishes every artifact: the GUI (`.deb`, `.rpm`, AppImage, via `cargo tauri build`) and the CLI (`.deb`, `.rpm`, tarball). Cutting a release is `git tag v0.1.0 && git push origin v0.1.0`; the workflow refuses to build if the tag disagrees with the workspace version, and it publishes as a **draft** so the assets can be looked at before anything goes public. `workflow_dispatch` runs the whole pipeline without publishing, which is how you rehearse a release without burning a version number.

**Bumping the version — always use `scripts/bump-version.sh`, never edit by hand.** The version is declared in six files that must never disagree (`Cargo.toml`, the `bulwark-core` path-dep pin in `crates/bulwarkctl/Cargo.toml`, both `package.json`s, `apps/bulwark-app/src-tauri/tauri.conf.json`, and the `getVersion` mock in `apps/bulwark-app/src/mocks/tauri/app.ts`), plus `Cargo.lock`. Miss one and you ship a `0.5.0` package whose `--version` prints `0.4.0`, or a tag CI rejects. The script sets all of them and syncs `Cargo.lock`:

```bash
scripts/bump-version.sh 0.5.0     # set every declaration + sync Cargo.lock
scripts/bump-version.sh --check   # verify they already agree (use in CI / before tagging)
```

Then commit (`chore(release): 0.5.0`) and tag. If a new file starts carrying the version, add it to the script's file list — that list is the single source of truth for what a bump touches.

Built on `ubuntu-22.04`, deliberately not `ubuntu-latest`: the oldest glibc linked against becomes the oldest distro the artifacts run on, and `ubuntu-latest` silently raises that floor whenever GitHub re-points it.

### Architectures (x86_64 + aarch64)

Every artifact — CLI and GUI, `.deb`/`.rpm`/AppImage/tarball — ships for **both x86_64 and aarch64**, built **natively** on `ubuntu-24.04` and `ubuntu-24.04-arm` runners (free for public repos). Nothing in the Rust source is arch-specific: no `#[cfg(target_arch)]`, no `std::arch`, no SIMD, no asm, and the only native dependency is `libsqlite3-sys`, which compiles vendored SQLite C. `remote.rs`'s SSH push path was already arch-generic (`compatible_uname_m`, fail-closed on an unknown arch).

**Native, not cross-compiled.** `src-tauri/build.rs` *used to* stage the CLI sidecar from `target/<profile>/bulwarkctl` — the host target dir — so a `cargo build --target aarch64-…` would silently stage an **x86_64** binary under an aarch64 triple's name. That is fixed: it now honours `CARGO_TARGET_DIR` and checks `target/<triple>/<profile>` **first**, so a cross-build can no longer fall back to a stale host binary. Native runners remain the right choice anyway — they are free for public repos, and they are the only way to launch-test a GUI honestly — but the cross-compile footgun itself is gone rather than merely avoided. Likewise the GUI launch test runs on real arm64 hardware rather than qemu: a WebKit GUI under emulation is minutes-per-frame slow and fails on graphics paths no user touches, so a red result would carry no information.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietanhdev/bulwark](https://github.com/vietanhdev/bulwark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
