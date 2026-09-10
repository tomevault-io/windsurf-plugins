---
trigger: always_on
description: validates this consistency check and will fail otherwise.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in this repository.

## What this project is

`tg-ws-proxy-rs` is a Rust port of [Flowseal/tg-ws-proxy](https://github.com/Flowseal/tg-ws-proxy): a local
MTProto proxy that tunnels Telegram Desktop traffic through WebSocket connections to Telegram's
data centers. It exists for networks where raw TCP to Telegram is blocked but WebSocket/HTTPS to
`web.telegram.org` still works (common in Russia and other censored networks).

At a high level: a Telegram client (Desktop, or any MTProto-speaking client) connects to this
proxy over plain MTProto (optionally disguised as FakeTLS). The proxy de-obfuscates the MTProto
transport frame, re-encrypts it for the target DC, and forwards it — preferring a WebSocket
tunnel to `wss://kwsN.web.telegram.org/apiws`, with Cloudflare-proxy, Cloudflare-Worker, upstream
MTProto proxy, and raw TCP as successive fallbacks.

## Repository layout

```
src/
  main.rs              Thin CLI wrapper: clap parse, tracing, then server::run
  server.rs            Process-level bind / banner / accept loop (shared by the binary and embedders)
  config.rs            clap-derived Config struct; all CLI flags + TG_* env var fallbacks
  proxy.rs              Core per-connection logic: client handshake, DC routing, WS/CF/TCP fallback chain
  crypto.rs             MTProto obfuscated-transport crypto (AES-256-CTR key derivation, secret layout)
  faketls.rs            0xee FakeTLS camouflage: fake TLS 1.3 handshake for inbound + upstream proxies
  splitter.rs            Splits/reassembles MTProto transport frames from WebSocket message boundaries
  ws_client.rs           WebSocket client that dials Telegram DC endpoints (kwsN.web.telegram.org)
  pool.rs                Pre-warmed pool of idle WebSocket connections per DC (cuts handshake latency)
  check.rs               `--check` connectivity tester for CF domains and upstream MTProto proxies
  limits.rs              Connection cap derived from the process file-descriptor budget
  default_domains.rs      Fetches + deobfuscates the community CF-proxy domain list from GitHub
  runtime.rs              Shared runtime state (outbound connector, fronting window, DC metadata)
  outbound/               Outbound TCP connector: HTTP/SOCKS5(h) proxy support, NO_PROXY matching
  default_domains/http.rs  Minimal HTTPS GET used only to fetch the default domain list

tests/                   Integration tests (one file per subsystem, mirrors src/ module names)
tests/common/mod.rs      Shared integration fixtures (fake HTTP CONNECT proxy, proxy-connection driver)
docs/                    User-facing guides. README stays an overview and links here rather than growing:
                         Fallbacks.md (routing tiers), Building.md (cross-compiling, UPX), Deployment.md
                         (Docker, OpenWrt, env vars), CfProxy.md + CfWorker.md (Cloudflare setup),
                         Android.md (Compose app + NDK build)
android/                 Jetpack Compose app (Gradle catalog + build-logic convention; see docs/Android.md)
crates/android-jni/      JNI start/stop + log callback cdylib; built only by the Android Gradle task
```

Modules whose *private* internals need testing keep a `#[cfg(test)] mod tests;` in a sibling
`src/<module>/tests.rs` file (see `proxy/`, `ws_client/`, `limits/`, `default_domains/`).  Anything
reachable through `src/lib.rs` is tested from `tests/` instead.

`src/lib.rs` re-exports the crate's internals so integration tests in `tests/` can exercise them
directly; almost all logic lives in the library, `main.rs` is a thin binary wrapper.

## Build, test, lint

```bash
cargo build                        # debug build
cargo test                         # unit + integration tests (tests/*.rs)
cargo clippy --all-targets         # CI runs this; keep it clean of new warnings
cargo fmt                          # run before committing — CI does not auto-format
cargo build --release              # CI also does a release build with LTO (see Cargo.toml profile)
```

**Those cover the root package only.** `Cargo.toml` sets `default-members = ["."]`, which keeps
`crates/android-jni` out of every bare `cargo build` / `test` / `clippy` at the repository root —
a `cdylib` in the default set made every desktop and Docker build link an `.so` nobody loads. So
a change to the JNI shim is compiled by none of the commands above. Reach it by name:

```bash
cargo fmt --all                                # unlike the others, this does cover crates/
cargo clippy -p tg-ws-proxy-jni --all-targets  # host: lib.rs only, `mod android` is cfg'd out
cd android && ./gradlew :app:cargoNdk          # the real compile: cargo build -p … --target <triple>
```

Note what the host commands do *not* prove. `crates/android-jni/src/android.rs` sits behind
`#[cfg(target_os = "android")]`, and its dependencies behind a matching
`[target.'cfg(…)'.dependencies]` table, so off Android the crate is an empty cdylib:
`cargo clippy -p tg-ws-proxy-jni` type-checks none of the shim. Only the cross-compile does, and
that needs an NDK.

There is no host-runnable test for the shim either. Its behaviour is covered by the Android
instrumentation suite in `android/app/src/androidTest/` — the JNI contract tests and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valnesfjord/tg-ws-proxy-rs](https://github.com/valnesfjord/tg-ws-proxy-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
