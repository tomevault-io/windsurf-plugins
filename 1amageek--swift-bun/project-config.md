---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Build & Test

```bash
# Build
swift build

# Run all tests (with timeout)
swift test

# Run a bounded test and kill stale helpers on timeout
scripts/swift-test-timeout.sh 30 --filter "BunProcess"

# Repeat a suite under hang guard
scripts/swift-test-hang-guard.sh --repeats 3 --timeout 30 --build-timeout 120 -- \
  --filter "BunProcessAsyncTests"

# Run the 0.1.0 release verification lane
scripts/release-check-0.1.0.sh

# Check for sync shutdown in deinit before broad runs
scripts/check-sync-shutdown-in-deinit.sh Sources Tests

# Run a specific test suite
swift test --filter "BunProcess"

# Run a single test by name
swift test --filter "setTimeout"
```

Network roundtrip tests (`FetchRoundtripTests`) hit `httpbin.org` and require internet access.

### Hang-resistant runtime tests

`BunProcess`-backed tests are not safe to parallelize purely with `.serialized`, because Swift Testing only serializes within a suite. Cross-suite runtime tests must go through `TestProcessSupport` helpers so they share the same `RuntimeTestGate`.

- Use `TestProcessSupport.withLoadedProcess(...)` for library-mode tests.
- Use `TestProcessSupport.run(...)` for `BunProcess.run()` in tests.
- Wrap local HTTP/WebSocket server helpers in `TestProcessSupport.withExclusiveRuntimeAccess(...)` when they participate in the same test flow as `BunProcess`.
WebSocket tests use a local NIO-based server and do not require internet access.

### Test bundle regeneration

The test bundle `Tests/BunRuntimeTests/Codex.bundle.js` is a resource copied into the test target via `Package.swift`. To regenerate it from `Fixtures/Codex-test/`:

```bash
cd Fixtures/Codex-test && npm install
npx esbuild index.js --bundle --platform=node --target=es2020 --format=cjs \
  --external:node:* --outfile=Codex.bundle.js
cp Codex.bundle.js ../../Tests/BunRuntimeTests/
```

The ESM transformer bundle `Sources/BunRuntime/Resources/esm-transformer.bundle.js` is regenerated from `Fixtures/esm-transformer/`:

```bash
cd Fixtures/esm-transformer && npm install
npx esbuild index.js --bundle --platform=node --target=es2020 --format=cjs \
  --outfile=esm-transformer.bundle.js
cp esm-transformer.bundle.js ../../Sources/BunRuntime/Resources/
cp esm-transformer.bundle.js ../../Tests/BunRuntimeTests/
```

The Web API polyfills bundle `Sources/BunRuntime/Resources/polyfills.bundle.js` is regenerated from `Fixtures/polyfills/`:

```bash
cd Fixtures/polyfills && npm install
npx esbuild index.js --bundle --platform=node --target=es2020 --format=cjs \
  --outfile=polyfills.bundle.js
cp polyfills.bundle.js ../../Sources/BunRuntime/Resources/
cp polyfills.bundle.js ../../Tests/BunRuntimeTests/
```

## Architecture

Primary architecture spec: `Docs/RuntimeArchitecture.md`

JavaScript placement rules are also defined in `Docs/RuntimeArchitecture.md` under `JavaScript source placement`.
JavaScript loading and resource layout are defined in `Docs/JavaScriptLoading.md`.

swift-bun provides a Bun-compatible JavaScript runtime for iOS/macOS by wrapping JavaScriptCore with Node.js/Bun polyfills. It uses SwiftNIO for the event loop (NIOCore + NIOPosix).

### Execution model: BunProcess

`BunProcess` is the sole execution model. Configuration is provided at `init`, execution via `load()` or `run()`.

```swift
BunProcess(bundle: URL?, arguments: [String], cwd: String?, environment: [String: String])

.load()  // Library mode — then evaluate(js:) / call(), and must be paired with shutdown()
.run()   // Process mode — blocks until exit and shuts down before returning
.shutdown() // Explicit cleanup for library mode
```

All JSContext access is serialized on a dedicated NIO EventLoop thread, guaranteeing thread safety.

```
BunProcess (final class, Sendable)
├── Configuration (immutable): bundle, arguments, cwd, environment
├── EventLoop thread (NIO MultiThreadedEventLoopGroup, 1 thread)
│   ├── JSContext (all access pinned to this thread)
│   ├── Web API polyfills (polyfills.bundle.js)
│   ├── ModuleBootstrap polyfills (require, Node.js modules, Bun APIs)
│   └── Host-backed bridges:
│       ├── setTimeout/setInterval → eventLoop.scheduleTask
│       ├── fetch (__nativeFetchStream) → URLSession streaming bridge + eventLoop.execute
│       ├── globalThis.WebSocket → URLSessionWebSocketTask bridge
│       ├── process.stdout.write → stdout AsyncStream
│       ├── process.stdin → sendInput() from Swift
│       ├── process.exit → resolveExit()
│       ├── console.log → output AsyncStream
│       ├── node:net / node:http server → NIO bridges
│       └── Web Crypto / dns / zlib → native bridges
├── Lifecycle (state machine + boot barriers + explicit shutdown)
└── ESM transformer (es-module-lexer WASM, temporary JSContext)
```

### Polyfill layers

JSCore's `evaluateScript()` provides only ECMAScript language features (Promise, Symbol, BigInt, etc.). All platform APIs are polyfilled in three layers:

```
Layer 0: polyfills.bundle.js + runtime scripts ← Web APIs (JS-owned semantics)
Layer 1: ModuleBootstrap                        ← Node.js globals + modules (Swift strings)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/swift-bun](https://github.com/1amageek/swift-bun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
