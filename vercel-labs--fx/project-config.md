---
trigger: always_on
description: These instructions apply to work under `sdk/` and supplement the repository-level `AGENTS.md`.
---

# WebAssembly SDK instructions

These instructions apply to work under `sdk/` and supplement the repository-level `AGENTS.md`.

This file is maintainer guidance, not consumer documentation. Keep setup and the supported public surface in `sdk/README.md`. Keep exact API shapes in `sdk/fx-sdk.js` and its tests. Add details here only when they help an agent choose the right owner, preserve a non-obvious invariant, or run the right proof. The native Node-API architecture and security model are documented in `sdk/NAPI.md`.

## Start from the correct owner

The SDK has two WebAssembly surfaces and one shared JavaScript host layer:

| Concern | Source of truth |
| --- | --- |
| Shared loader, host imports, adapters, and public JavaScript exports | `sdk/fx-sdk.js` |
| Headless ACP composition for `fx-core.wasm` | `src/wasm_core_main.zig` |
| Interactive terminal entry point for `fx-term.wasm` | `src/wasm_term_main.zig` and `runWasmTerminal` in `src/main.zig` |
| Native and WebAssembly capability policy | `src/core/hosts/runtime_profile.zig` |
| Host-backed terminal session persistence | `src/core/app/app_session_runtime.zig` and `sdk/fx-sdk.js` |
| Browser workspace contract and `shell.run` bridge | `src/core/hosts/js_host_workspace.zig` and `src/tools/shell/browser_shell.zig` |
| Browser device login, OAuth session persistence, and URL opening | `src/core/auth/js_host_auth.zig`, `src/core/auth/oauth_session.zig`, and `src/core/hosts/js_host_url_opener.zig` |
| WASI target, optimization mode, threading, and artifact names | `build.zig` |
| Core browser fixture and its automation contract | `sdk/index.html` and `sdk/tests/test-core-browser.mjs` |
| Terminal fixture and static packaging contract | `sdk/term-demo.html` and `sdk/scripts/package-term-demo.mjs` |
| Commands required by CI | `.github/workflows/ci.yml` |

Do not treat the demos or this file as the implementation contract. When prose and behavior disagree, establish the current behavior from code and focused tests before changing either.

## Preserve these invariants

- Keep `sdk/fx-sdk.js` a dependency-free ECMAScript module. A bundler, framework, or runtime package must not become necessary to load the SDK.
- Keep the core and terminal surfaces distinct. `fx-core.wasm` starts the headless ACP server; `fx-term.wasm` starts the interactive terminal. Shared loader changes must be validated against both.
- Detect JavaScript Promise Integration (JSPI) by capability through `supportsJspi()`. Do not replace feature detection with browser or version sniffing. Keep loader errors, demo fallback states, and the compatibility statement in `sdk/README.md` consistent.
- Treat JavaScript host stores as durable contracts. Session and OAuth snapshots are opaque bytes with optimistic revisions. Preserve `FX_SESSION_REVISION_CONFLICT` and `FX_OAUTH_SESSION_REVISION_CONFLICT`. Persist configuration only after fx accepts it, and do not collapse prompt-history outcomes into generic success.
- Preserve cancellation and lifecycle behavior. Fetch cancellation must reach the host `AbortSignal`; terminal subscriptions must be released exactly once; `abort()` must settle `exited` and must not leave input or resize listeners attached.
- The WebAssembly runtime is not the native runtime. Keep native tools disabled. The optional workspace host may expose only completion-only `shell.run` through its typed boundary and permission policy. Its schema is exactly `{ action: "run", command }`; native profiles, TTYs, and managed running handles are unavailable. Any additional capability requires its own typed host boundary, permission review where applicable, and coverage on the affected surface.
- Keep workspace version 1 constrained to an ephemeral, non-git workspace whose normalized `cwd` equals `root`. Preserve command and output limits, the 30-second maximum deadline, and Ctrl+C cancellation through the shared host-effect abort path.
- `window.__fxCoreTest` and `document.body.dataset.state` are test interfaces for the core debugger. If either changes intentionally, update the browser test in the same change.
- The live demos may pass a locally stored credential into the WebAssembly environment. Never print, serialize into artifacts, or add test assertions containing that credential.

## Match the proof to the change

| Change | Minimum focused proof |
| --- | --- |
| Public export, shared loader, WASI import, or JSPI gate | Build and test both surfaces |
| `createFxAgent()`, ACP translation, core session persistence, streaming, or cancellation | Core build, core Node tests, and the browser test when browser behavior is involved |
| Live Gateway request or model-catalog translation | Core tests plus the opt-in live smoke test when a credential is available |
| Terminal adapter, input encoding, resize, cleanup, config, or prompt history | Terminal build and the headless terminal suite |
| Terminal session persistence or browser device login | Terminal build plus `sdk/tests/test-term-session-resume.mjs` or `sdk/tests/test-term-login.mjs` |
| Browser workspace metadata, permissions, execution, limits, or cancellation | Terminal build plus `sdk/node/test-term-workspace.mjs` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/fx](https://github.com/vercel-labs/fx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
