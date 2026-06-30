---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Pingap is a Cloudflare-Pingora-based reverse proxy. The binary lives in `src/`; all reusable logic is split across `pingap-*` workspace crates. MSRV is `1.88.0` (Rust edition 2024). Pingora is pinned to `0.8.0` and only the `lb`/`openssl`/`cache` features are enabled.

## Common commands

```bash
# Bacon-driven dev loop (uses --autoreload + admin UI on :3018)
make dev                 # bacon run -- --features=full -- -c=... --admin=...

# Lint (CI gate — runs typos + clippy --features=full --all-targets -- -D warnings)
make lint

# Format
make fmt

# Full test suite (requires the `full` feature set)
make test                # cargo test --workspace --features=full

# One package / one test (use cargo directly, not make)
cargo test -p pingap-proxy
cargo test -p pingap-core util::tests::test_now_ms -- --nocapture

# Benchmarks (criterion)
make bench               # workspace-wide
make bench-all           # explicit list (pingap-core, pingap-logger, pingap-location)

# Release builds — see Makefile for the matrix
make release             # default features
make release-full        # tracing + imageoptim
make release-perf        # release-perf profile, includes pyroscope agent

# Web admin assets (rust-embed'd into the binary at build time)
make build-web           # cd web && npm install && npm run build, then cp dist ../

# Pre-commit hook (runs `make lint`)
make hooks               # cp hooks/* .git/hooks/
```

Bacon shortcuts: `bacon` (check), `bacon clippy-all`, `bacon test`, `bacon test -- some::path`, `bacon nextest`, `bacon doc-open`.

## Architecture

The dependency layering (see `docs/modules.md` for the full mermaid graph) is roughly:

```
util -> core -> {discovery, config, logger, location, cache, certificate, upstream, plugin, health}
                        |
                        v
              acme, performance, otel, sentry, pyroscope, imageoptim, webhook
                        |
                        v
                     proxy <- top binary (`src/main.rs`)
```

- `pingap-core` — `HttpResponse`, `Ctx`, plugin traits, coarse clock helpers (`now_sec`/`now_ms`/`real_now_ms`), `BackgroundTaskService`, `ClockUpdaterService`. Every other crate depends on it.
- `pingap-config` — `PingapConfig` model plus storage backends (`file_storage.rs`, `etcd_storage.rs`) chosen at runtime by URL prefix (`file://`, `etcd://`). Supports TOML, HCL (`hcl.rs`), and KDL (`kdl.rs`) input formats.
- `pingap-proxy` — implements pingora's `ProxyHttp`. The request lifecycle in `pingap-proxy/src/server.rs` calls (in order) `early_request_filter` -> `request_filter` -> `proxy_upstream_filter` -> `upstream_request_filter` -> `upstream_response_filter` -> `logging`. Each step matches the `PluginStep` enum in `pingap-core/src/plugin.rs` (`EarlyRequest`, `Request`, `ProxyUpstream`, `UpstreamResponse`, `Response`); a plugin runs at most one step per request.
- `pingap-plugin` — built-in plugins. Add new ones by implementing the `Plugin` trait from `pingap-core` and registering them via the plugin factory.
- `pingap-upstream` — pingora `Backends` + load-balancing wiring; gets its backend set from `pingap-discovery` (static / DNS / Docker labels / transparent) and `pingap-health` for active checks.
- `src/main.rs` — argument parsing, config bootstrap, daemonization, server assembly. The `src/process/` and `src/plugin/` modules handle hot reload + the admin plugin.
- `build.rs` — uses `vergen = "9.1.0"` + `vergen-git2 = "9.1.0"` to embed `VERGEN_GIT_SHA` into the binary's `--version`. **Both crates must stay on matching majors**; if you see `Add` trait-bound errors from `vergen_lib`, the lockfile has pulled mismatched versions — refresh it.
- `examples/` — working configs to copy from: `api-gateway`, `grpc-web`, `static-serve`, `transparent-proxy`, `web-socket`.

### Hot reload vs auto-restart

`src/main.rs` branches on `pingap_config::ConfigManager::support_observer()`. **etcd** returns `true` and pushes changes via a `WatchStream` (`pingap-config/src/etcd_storage.rs`) wired through `new_observer_service`. **File** storage returns `false` and is polled by `new_auto_restart_service` (`src/process/auto_restart.rs`) on a fixed interval. Both feed the same `reload_handle` — the difference is only the delivery mechanism. `--autoreload` keeps the process and swaps config in place; `--autorestart` performs a zero-downtime graceful restart for changes that need a fresh listener.

### Daemonization and the coarse clock

Pingora forks inside `Server::run_forever()` for daemon mode. **`fork()` only carries the calling thread**, so any `std::thread` (including coarsetime's background updater) started before the fork is gone in the child.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vicanso/pingap](https://github.com/vicanso/pingap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
