---
trigger: always_on
description: This repo is the single source of truth for the TrUAPI protocol. It vendors `dotli` as a git submodule at `hosts/dotli/`.
---

# Repository agent guidance

This repo is the single source of truth for the TrUAPI protocol. It vendors `dotli` as a git submodule at `hosts/dotli/`.

## Layout

```
rust/crates/
  truapi/                Rust trait + type definitions for protocol versions v0.1 and v0.2
                         (canonical), plus the runtime hosts implement (default `runtime`
                         feature); ships as WASM (browser/node); its `platform` module
                         holds the host syscall traits (storage, navigation, consent, ...)
  truapi-codegen/        rustdoc JSON → TypeScript client + Rust dispatcher
  truapi-macros/         #[wire_trait(id = N)] and #[wire(id = N)] proc-macros;
                         #[sso_service] for truapi's inter-host SSO protocol
                         One implementation module per macro; lib.rs holds entry points
  truapi-provider/       network provider backends (WebSocket RPC or smoldot light-client);
                         its `platform` module holds the chain-access traits
  truapi-verifiable/     ring-VRF operations over `verifiable`; a lazily loaded WASM module in the browser
  truapi-host-cli/       CLI pairing/signing hosts; Bun scripts share the container web API gates
js/packages/
  truapi/                  @parity/truapi TS package; generated TS lives under ignored paths
  truapi-host/            @parity/truapi-host: WASM-backed host runtime. Subpath entries:
                          `.` (shared host types), `/web` (iframe + Web
                          Worker), `/worker-runtime` (Worker entry), and the
                          test host `/testing` (createMockHost),
                          `/testing/playwright` (fixture), `/testing/server`
                          (node server), `/testing/client` (no-iframe client),
                          `/testing/dev-accounts`, `/testing/host-page`.
                          Two WASM bundles (gitignored) under dist/wasm/, built
                          via `make wasm`: `web/` is the production browser host
                          and `testing/` adds the `wasm-signing-host` and
                          `test-host` Cargo features the test host needs
  truapi-debugger/        @parity/truapi-debugger (published to npm): the debugger.
                          Owns all decoding of the wire frames the Rust host tap
                          (truapi's DebugSink) streams out, and decodes
                          every frame by default (no denylist, no reveal toggle).
                          Holds the trace, envelope-decode, and value-decode
                          engines, the shared view model + renderers, and two
                          mounts over them: server.ts (standalone WS+HTTP app on
                          127.0.0.1:9231 that hosts dial into, `npm run serve`;
                          endpoints /, /op-list, /op, /view, /channels, /stats,
                          /traces, /frame) and in-app.ts (createInAppDebugger:
                          same-page host, no server, no dial). @parity/truapi has
                          no debug seam. Where the app ultimately lives is still
                          an open decision.
js/container/              Shared TS lockdown container for native web views and CLI dev; scripts reuse its web API gates
                           `npm run build` bundles it into ios/truapi-host/Sources/TrUAPIHost/Resources/
ios/truapi-provider/       TrUAPIProvider Swift package (chain transport over UniFFI);
                           second product of the root Package.swift, released on its
                           own tag (@parity/ios-provider@<v>) via its scripts/
android/truapi-host/       truapi-host-android AAR (bindings + Kotlin shell + per-ABI
                           cdylib), published to GitHub Packages by release-android;
                           include `@parity/android-host <version>` in the `release:`
                           PR title
android/truapi-provider/   truapi-provider-android AAR; bundles the cdylib the same way,
                           so consumers need no Rust toolchain
ios/truapi-host/           TrUAPIHost Swift package over the truapi UniFFI core;
                           SPM manifest at the repo root (Package.swift), rebuild via
                           ios/truapi-host/scripts/rebuild.sh
playground/                Next.js interactive playground; deploys to the truapi-playground dotNS label
hosts/ios/                 iOS host app; resolves the core from this tree
hosts/android/             Android host app
hosts/imports.json         source repository and imported revision per host,
                           read and updated by scripts/refresh-host-import.sh
hosts/dotli/               dotli submodule
docs/                      design docs, RFCs, feature proposals
scripts/codegen.sh         regenerate the TS client from the Rust crate
scripts/battery.sh         run the generated battery against both headless CLI host roles,
                           plus the Pocket phase a Worker execution serves
scripts/bundle-size.mjs    measure the JS/WASM of the asset groups given (raw, gzip, brotli);
                           .github/actions/bundle-size takes them as `assets`, stores

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paritytech/trinity-user-agents](https://github.com/paritytech/trinity-user-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
