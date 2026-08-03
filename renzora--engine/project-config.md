---
trigger: always_on
description: > **This file is the authoritative guide for working in this repository.** It
---

# CLAUDE.md — Renzora Engine

> **This file is the authoritative guide for working in this repository.** It
> overrides assumptions and habits from other Rust/Bevy projects. Read it before
> building, testing, writing plugins, extending the scripting API, or editing
> docs. When something here conflicts with what "usually" works, this file wins.

Renzora is a Bevy-based game engine + editor. The workspace is ~150 `renzora_*`
crates plus a small set of vendored/forked Bevy ecosystem crates. The engine
ships as a **single binary** that runs as the editor when the editor bundle is
present beside it, and as the shipped game/server when it isn't.

---

## 1. The `renzora` CLI

All real work goes through the `renzora` CLI, which drives a pinned Docker
container. It is a **separately published tool**, not part of this workspace.

- Install: `cargo install renzora`
- crates.io: <https://crates.io/crates/renzora>
- Source: <https://github.com/renzora/cli>

| Command | What it does |
|---|---|
| `renzora init` | Pull/build the host toolchain image + create/start its container (idempotent) |
| `renzora check` | `cargo check` in the linux container (clippy-style gate) |
| `renzora test [args]` | Run the test suite in the linux container (no args = workspace suite) |
| `renzora build [platforms]` | Cross-build for one or more platforms (no args = all) |
| `renzora run` | Build for this host and launch it (editor by default) |
| `renzora add <name>` | Scaffold a new plugin crate |
| `renzora remove <name>` | Delete a plugin crate |
| `renzora shell` | Interactive shell inside the linux container |
| `renzora destroy` | Remove this checkout's containers + build-cache volumes |
| `renzora prune` | Remove this checkout's stale (non-current) toolchain images |
| `renzora new` | Create a new project by cloning the engine |

**Split toolchain images.** The toolchain is one shared base image
(`base`: rust + Linux deps + LLVM-19) plus one image per platform built
`FROM` it (`linux`, `windows`, `macos`, `ios`,
`android`, `wasm`). `renzora run` pulls only the host platform
image; `renzora build` (no args) pulls all; `renzora build windows` pulls only
Windows. Each platform runs in its own container; Linux-native ops (`test`,
`check`, `shell`, `clean`, `add`/`remove`, `upx`) use the linux container. Tags
are content hashes: `baseTag = sha256(docker/base/Dockerfile)` and
`<plat>Tag = sha256(baseTag + docker/<plat>/Dockerfile)`, so a base edit
cascades to every platform while a platform edit moves only its own tag. Stale
tags are pruned automatically on update.

If you need the user to run an interactive/auth command, suggest they prefix it
with `!` in the prompt so its output lands in the session.

---

## 2. Building & testing — Docker preferred, native supported

**Prefer Docker via the `renzora` CLI for building and testing.** It's the
canonical environment, the only path for cross-platform/release builds, and what
CI runs — so it's the ground truth when verifying. A **native** (no-Docker) build
of the host platform is also supported via `cargo renzora` (see below), for local
convenience; reach for it when a contributor wants to build/run on their own
machine without Docker, but fall back to Docker whenever results must match the
canonical env or another platform is involved.

A note on the old "native can't link" claim: the shared `renzora` dylib plus the
full plugin set exceeds the **65,535 exported-symbol cap** of the Windows PE
format, which MSVC `link.exe` refuses. That is *not* a blocker, because
`.cargo/config.toml` already pins the linker to **`rust-lld`** for
`x86_64-pc-windows-msvc` (host and container alike), and rust-lld has no such cap.
So native links succeed; we simply never use `link.exe`. So:

- ✅ `cargo check` natively / via the editor — the fast local gate while editing
  (doesn't link).
- ✅ `cargo renzora` — native build + stage + run on the **host platform** (an
  `xtask` that mirrors the container's `build-all.sh` staging; `rust-toolchain.toml`
  pins rustc so it matches the images). Convenient, but host-only.
- ✅ `renzora check`, `renzora test`, `renzora build`, `renzora run` — the
  canonical builds inside the container; **required** for cross-platform/release
  and for reproducing CI. **Prefer these when verifying.**
- ❌ Don't "fix" a perceived link error by stripping the `dylib` crate-type or
  disabling `prefer-dynamic` — the shared `bevy_dylib`/`renzora` dylib is
  load-bearing for the plugin ABI (§3).

Pinned toolchain — **Rust 1.95.0**, **Bevy 0.19**. The Rust version lives in TWO
files kept in lockstep: `docker/base/Dockerfile` (`FROM rust:1.95.0`, the
container) and `rust-toolchain.toml` (native `cargo renzora` / `cargo check`); a
bump must edit both. The base image is the foundation every platform image builds
`FROM`, so a container bump cascades to all platforms — see §3. CI
(`.github/workflows/test.yml`) runs `cargo test` + `cargo clippy -D warnings` in
the `base` image, excluding the vendored `bevy_*` / `vleue_navigator` crates. Keep
clippy green; the vendored crates must stay excluded.

---

## 3. Plugin ABI — the `bevy_dylib` it links

Community/distribution plugins are `dlopen`'d at runtime and share **one

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renzora/engine](https://github.com/renzora/engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
