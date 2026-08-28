---
trigger: always_on
description: Guidance for AI coding agents working in this repo. User-facing docs, code comments, and UI strings are written in Chinese — match that.
---

# AGENTS.md

Guidance for AI coding agents working in this repo. User-facing docs, code comments, and UI strings are written in Chinese — match that.

## Layout

Five **standalone Rust crates, no root Cargo workspace** — Cargo commands fail from the repo root with "could not find Cargo.toml"; always run them inside a crate dir (each has its own `target/`):

- `lib/` — shared protocol core, compiled by BOTH native clients and the Cloudflare Worker
- `client_cli/` — CLI client
- `client_tauri/` — GUI client: React 19 + Vite frontend (`src/`), Tauri 2 backend (`src-tauri/`)
- `server-rs/` — Cloudflare Worker (Rust → wasm32 via `worker-build`; worker crate 0.8 + axum)
- `lib_test/` — E2E harness **binary** (`cargo run`, never `cargo test`)

Leave alone: `client_tauri/src-tauri/gen/`, `icons/`, `image/`, `logs/`, `lib_test/src/latest_test.log`, per-crate `Cargo.lock`.

CLI and GUI deliberately share one config/CA directory (`app_data_dir/com.zz.freeproxy`) — the identifier is hardcoded in both `client_cli/src/config.rs` (`IDENTIFIER`) and the Tauri config; changing either desyncs the two clients.

## Directory structure

Annotated map of source dirs (build/runtime dirs like `target/`, `.wrangler/`, `icons/`, `image/`, `logs/` are omitted):

```
├── package.json                  # root npm scripts (server-dev/deploy, client-dev, test-e2e)
├── deny.toml                     # cargo-deny shared config (all 4 crates read it)
├── .github/workflows/release.yml # tag-triggered release CI (desktop + CLI + Worker zip; Android job commented out)
├── lib/src/                      # shared protocol core — compiles for native AND wasm32
│   ├── aead.rs                   #   AEAD ciphers: AES-GCM/GCM-SIV, ChaCha20-Poly1305/XChaCha20
│   ├── algo.rs                   #   compressor × AEAD negotiation + URL contract /api/{version}/{target}
│   ├── base.rs                   #   base encodings (base64 / z85 / base91)
│   ├── compress.rs               #   zstd / gzip / lz4
│   ├── ecc.rs                    #   ECDSA/ECDH (k256/p256/p384)
│   ├── frames.rs                 #   binary frame stream: [u32 BE len | payload], zero-length frame = EOS
│   ├── hash.rs                   #   sha1/sha2/blake3 wrappers
│   ├── http.rs                   #   httparse header parsing + UrlBuilder (zero-copy)
│   ├── kdf.rs                    #   PBKDF2 / scrypt / HKDF wrappers
│   ├── tool.rs                   #   derive_keys (auth_key+domain), time-window token auth, XOR obfuscation
│   ├── ws.rs                     #   RFC 6455 frames + WsTunnelMsg (shared by client & worker)
│   ├── proxy/                    #   CLIENT ONLY (feature "client"): local HTTP proxy
│   │   ├── mod.rs                #     ProxyConfig / Shared / Proxy lifecycle facade
│   │   ├── connection.rs         #     connection dispatch (plain HTTP vs CONNECT)
│   │   ├── relay.rs              #     forwarding engine (serve loop, keep-alive via EOS)
│   │   ├── body.rs               #     request-body boundary parsing
│   │   ├── client.rs             #     upstream reqwest clients (main/pref-IP/WS HTTP1.1)
│   │   ├── tls.rs                #     MITM TLS: self-signed CA + per-SNI leaf certs (moka cache)
│   │   └── ws.rs                 #     WS tunnel client side (RFC6455 parse/mask/reassemble)
│   └── speed_test/               #   CLIENT ONLY: 优选 IP two-phase speed test
│       ├── tcping.rs health.rs   #     phase 1 tcping → phase 2 Worker /health check
│       └── ip.rs                 #     Cloudflare IP candidate ranges
├── server-rs/
│   ├── wrangler.toml             # worker config; [dev] port=80; DO NOT touch compatibility_flags
│   ├── .dev.vars                 # gitignored dev secrets (key/domain); E2E harness rewrites it!
│   └── src/
│       ├── app.rs                #   axum router, Bearer auth middleware (±30 s window), routes
│       ├── proxy_http.rs         #   POST /api/{version}/{target}: streaming decrypt→fetch→re-encrypt
│       ├── proxy_ws.rs           #   GET /ws/{version}/{target}: upstream WS handshake + full-duplex relay
│       ├── subscribe.rs          #   GET /subscribe/{port}: Clash / sing-box / base64 subscription
│       └── lib.rs                #   worker entrypoint
├── client_cli/src/               # CLI client: main.rs (clap), run.rs (proxy loop), speed.rs,
│                                 # health.rs, ca.rs (cert install), subscribe.rs,
│                                 # config.rs (settings.json — shares app_data_dir with GUI)
├── lib_test/                     # E2E harness binary (`cargo run`, never `cargo test`)
│   ├── Cargo.lock
│   ├── Cargo.toml
│   └── src/
│       ├── cs.rs                 #   full-chain orchestrator (Worker + proxy + target site)
│       ├── main.rs               #   harness entrypoint
│       ├── test/
│       │   ├── base.rs           #     basic reachability cases (example.com http/https)
│       │   ├── http.rs           #     HTTP proxy test cases
│       │   └── mod.rs            #     shared proxied client + colored report runner
│       ├── util.rs               #   helpers
│       └── web.rs                #   local target site
└── client_tauri/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZEROLINGG/free-proxy](https://github.com/ZEROLINGG/free-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
