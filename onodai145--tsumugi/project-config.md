---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

tsumugi: a Misskey multi-column desktop client (Krile-like UX) built on Tauri v2. Rust core (`src-tauri/`) owns all Model-layer logic; the Svelte frontend (`frontend/`) is View/ViewModel only. Design docs live in `docs/design/` — `docs/design/misskey-multicolumn-client-design.md` is the authoritative design doc; if any other doc conflicts with it, the design doc wins. User-facing documentation lives in `docs/guide/user-guide.md`.

Frontend visual conventions (border-radius/font-size/icon-size/focus-visible scales) are codified in `docs/design/style-guide.md` — check it before adding new UI, don't invent one-off values (e.g. `rounded-[Npx]`).

## Commands

```sh
cargo tauri dev              # dev with hot reload — starts vite (127.0.0.1:5173) + the app together
cargo tauri build             # release build with frontend embedded (frontendDist), no dev server needed

cd src-tauri && cargo test    # Rust tests (real Misskey connectivity tests are #[ignore])
cd src-tauri && cargo test <test_name>   # run a single test
cd frontend  && pnpm check               # svelte-check + tsc (tsconfig.node.json)
cd frontend  && pnpm test                # Vitest unit tests

scripts/release.sh X.Y.Z      # bump version in package.json/Cargo.toml/tauri.conf.json/Cargo.lock, generate CHANGELOG.md, create release/vX.Y.Z branch + commit — never hand-edit these version fields with sed
```

Operation E2E tests (Playwright/tauri-driver against a throwaway Docker Compose Misskey instance) live in `e2e/` — see `e2e/README.md` for setup and run order, not covered here.

Never run `./target/debug/tsumugi` or `cargo run` directly — Tauri's debug build loads the frontend from the dev server (`devUrl` = `127.0.0.1:5173`); without vite running you get a connection-refused error. Always use `cargo tauri dev`.

`cargo tauri build --debug`'s bundler step can hang 40+ minutes on some dependency-tree changes (unrelated to compilation itself, which finishes in seconds). If you just need a runnable `target/debug/tsumugi` binary (e.g. for `tauri-driver`/E2E), use `cargo build` from `src-tauri/` directly, or `cargo tauri build --debug --no-bundle`.

E2E tests (`e2e/`) launch several background processes (`Xvfb`, `dbus-run-session`, `gnome-keyring-daemon`, the `tsumugi` binary under test) that can be orphaned if a run is killed abruptly. Clean these up by exact PID only (`ps aux` then `kill <pid>`) — never `pkill`/`killall` by name/pattern, since that can match unrelated real processes (e.g. a real browser) on the same machine.

On Linux/Wayland (Hyprland etc.), WebKitGTK's DMABUF renderer can conflict with wlroots compositors and crash rendering with `Gdk Error 71 (protocol error)`. `src-tauri/src/main.rs` sets `WEBKIT_DISABLE_DMABUF_RENDERER=1` by default to work around this; if that doesn't help, fall back to `GDK_BACKEND=x11 cargo tauri dev`.

### Android
Android build support exists (`src-tauri/tauri.android.conf.json`, `src-tauri/gen/android`). CI (`android-build` job in `.github/workflows/test.yml`) only verifies it compiles and links — no signing or artifact distribution:
```sh
cd src-tauri && cargo tauri android build --debug --target aarch64
```
Requires NDK r27c. The symlinks inside the NDK toolchain that `setup-ndk` extracts are relative, so they break if the extraction path changes — CI re-points them to absolute paths as a workaround (see the comment above the `android-build` job in `test.yml`).

If the build fails during Gradle configuration with `A problem occurred configuring project ':buildSrc'` and a cause like `IllegalArgumentException: 26.0.2` (or any JDK version string), it's not an NDK issue — Gradle 8.14.3's bundled Kotlin DSL compiler can't parse newer JDK version strings when evaluating `buildSrc`'s `build.gradle.kts`. Build with an older JDK, e.g. `JAVA_HOME=/usr/lib/jvm/java-21-openjdk cargo tauri android build --debug --target aarch64`.

## Architecture

### Rust ↔ TS boundary
`src-tauri/src/lib.rs` is the single source of truth for the command/event surface: `specta_builder()` registers every `#[tauri::command]` and every event type via `tauri-specta`. In debug builds, `run()` re-exports TS bindings to `frontend/src/bindings/tauri.gen.ts` on every launch; `cargo test`'s `generates_frontend_bindings` test also regenerates it and asserts serde's `camelCase` rename made it through and that account tokens are never exposed in generated types. When adding a command or event, register it in `specta_builder()`, not just the `tauri::Builder`.

### src-tauri/src layout
- `domain/` — normalized domain types shared across the app (Note, User, Account, Column, Reaction, Mute, ...), `specta::Type`-annotated for TS export.
- `api/` — REST client. **Hand-written, not generated** (see below) — thin typed wrappers per resource (`notes.rs`, `meta.rs`, `drive.rs`, ...) plus `normalize.rs` to convert raw responses into `domain` types. All Misskey REST calls are POST with the token embedded in the JSON body; that's centralized in `client.rs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onodai145/tsumugi](https://github.com/onodai145/tsumugi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
