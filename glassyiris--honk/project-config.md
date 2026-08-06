---
trigger: always_on
description: This file is written for AI coding agents that need to understand, build, test, and modify the project. It describes the actual layout and conventions observed in the repository (last verified against the tree on 2026-07-22).
---

# AGENTS.md — honk

This file is written for AI coding agents that need to understand, build, test, and modify the project. It describes the actual layout and conventions observed in the repository (last verified against the tree on 2026-07-22).

## Project overview

`honk` is a Rust transparent-proxy engine for Linux, **inspired by** [dae](https://github.com/daeuniverse/dae) (eBPF datapath and configuration surface) and [sing-box](https://github.com/SagerNet/sing-box) (outbound groups, multi-protocol dialers, Clash-compatible API). It is not a line-for-line port of either: the kernel path follows dae's TC + match_set + `dae0`/`daens` model, the userspace outbound/control stack follows sing-box-oriented designs.

- An eBPF transparent proxy engine (`honk-core`) intercepts traffic with eBPF TC redirect (no global `iptables` TPROXY rules), classifies it in eBPF, and relays it through proxy handlers in userspace.
- Shared configuration types and parsers (`honk-config`) parse the original dae `{ section { ... } }` configuration syntax — the primary and only documented config format.
- Status: **experimental alpha** (`v0.0.1-alpha`). Expect breaking changes.
- License: **GPL-3.0-only**. Repository: <https://github.com/Glassyiris/honk>
- Documentation: `README.md` / `README_CN.md` (bilingual overview, feature checklist, TODO list) and `doc/` — `design.en.md`, `configuration.en.md`, `components.en.md` (plus `.zh.md` translations), all currently in sync with the code.

## Repository layout

```text
.
├── Cargo.toml / Cargo.lock   # Workspace manifest (release + release-musl profiles)
├── Justfile                  # Day-to-day dev tasks (build, test, run, debug via clash API, cleanup)
├── README.md / README_CN.md  # Bilingual project overview
├── AGENTS.md                 # This file
├── LICENSE                   # GPL-3.0-only
├── config.dae                # Full-featured example config (production-leaning)
├── config.min.dae            # Minimal example (good for --mock-ebpf dev)
├── example.dae               # Annotated example (Chinese comments)
├── doc/                      # design / configuration / components docs (en + zh)
├── ci/                       # zigcc/zigcxx: zig cc/c++ wrappers for cross builds (strip CMake's clang-style --target from boring-sys ASM rules + rustc's aarch64 errata linker args; used by build-musl and the release workflow); zig-bindgen-env: derive BINDGEN_EXTRA_CLANG_ARGS from `zig cc -E -v` for cross bindgen
├── .github/workflows/        # release.yml: tag-triggered test + cross-build + GitHub Release
└── crates/
    ├── honk-config           # Config schema + dae-syntax parser + share links (workspace member)
    ├── honk-ebpf-common      # no_std shared eBPF/userspace types (workspace member)
    ├── honk-outbound         # Proxy handlers, groups, health checks (workspace member)
    ├── honk-core             # eBPF proxy engine, library + `honk-core` binary (workspace member)
    ├── honk-tool             # `honk-tool` CLI toolbox: `sub` subscription/node probing (workspace member)
    └── honk-ebpf             # Kernel eBPF programs (EXCLUDED from workspace, own Cargo.lock)
```

Notable absences (referenced by older docs but **not in this tree**): `Makefile`, `scripts/`, `Dockerfile`, `docker-compose.yml`, `plan.md`, `run_tests.sh`, `test-honk.sh`, `log/`, and the vendored reference checkouts (`honk/`, `outbound/`, `sing-box/` — these paths are `.gitignore`d). Consequences:

- `just run` and `just deploy` call `scripts/debug-local.sh` / `scripts/deploy-gateway.sh`, which do not exist here — use `just run-debug` / `just run-dae` instead.
- `just docker*` and the README Docker section reference a missing `Dockerfile` / `docker-compose.yml`.
- There are no root-only netns/podman test scripts in the checkout; all runnable tests are unprivileged.

## Technology stack

- **Language:** Rust, edition 2024 (workspace-wide, including the eBPF crate).
- **Async runtime:** Tokio (`full`).
- **eBPF:** userspace [aya](https://github.com/aya-rs/aya) 0.14 (optional `ebpf` feature in `honk-core`); kernel side `aya-ebpf` 0.2 targeting `bpfel-unknown-none` (nightly + `-Zbuild-std=core` + `bpf-linker`).
- **HTTP API:** axum 0.8 (with `ws`) + tower-http 0.7 (optional `clash-api` feature of `honk-core`, on by default).
- **QUIC:** quinn 0.11 (TUIC/Juicity/Hysteria2 outbounds, DoQ/DoH3 DNS); `h3`/`h3-quinn` for DoH3 only — Hysteria2 ships its own minimal HTTP/3+QPACK layer.
- **TLS:** [boring](https://github.com/cloudflare/boring) 5.x (BoringSSL) + tokio-boring for TCP TLS and — via the custom `quinn_proto::crypto` backend in `honk-outbound/src/quic_boring.rs` — for QUIC handshakes; webpki-root-certs for CA roots. rustls remains only as a **dev/test** dependency (loopback servers proving wire interop). boring-sys builds BoringSSL from source: requires `cmake` + a C compiler + `libclang` (bindgen).
- **Persistence:** rusqlite 0.40 (`bundled`) for the `cachedb` SQLite cache.
- **Serialization:** serde, toml 1, serde_json, serde_yaml.
- **Logging:** tracing + tracing-subscriber (`env-filter`, `json`); also `log`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Glassyiris/honk](https://github.com/Glassyiris/honk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
