---
trigger: always_on
description: A native SwiftUI macOS app that acts as a remote control for the Transmission BitTorrent daemon — a native equivalent to [transgui](https://github.com/transmission-remote-gui/transgui).
---

# TransmissionSwift — Agent Instructions

A native SwiftUI macOS app that acts as a remote control for the Transmission BitTorrent daemon — a native equivalent to [transgui](https://github.com/transmission-remote-gui/transgui).

This file gives AI coding agents (Claude, Cursor, Codex, Aider, etc.) the persistent context they need to be useful in this repo. Keep it concise.

## Read first

- `ARCHITECTURE.md` — durable architectural decisions and the rationale behind them.
- `doc/ui-buildout.md` — **the current implementation plan** (mock-first UI slices). Its "Picking up from a new session" section says exactly where work stands; start there.
- `doc/first-slice.md` — completed (2026-06-10); historical context only.

If anything in this file contradicts `ARCHITECTURE.md`, treat `ARCHITECTURE.md` as the source of truth and propose updating this file.

## Project shape (summary)

- **Platforms:** macOS only, min macOS 26, universal binary (arm64 + x86_64).
- **Module layout:** local Swift Packages under `Packages/`.
  - `TransmissionRPC` — wire protocol, Foundation only. No SwiftUI, no AppKit.
  - `TransmissionCore` — domain models, storage, services. Depends on `TransmissionRPC`.
  - App target (`TransmissionSwift/`) — SwiftUI views and view models only.
- **Concurrency:** `async`/`await` + `@Observable` macro. No Combine.
- **Storage:** JSON file for server profiles, Keychain for passwords, `@AppStorage` for UI prefs. No SwiftData on day one.

## Build & test commands

```bash
# Format Swift sources in-place (uses bundled swift-format).
swift format --in-place --recursive .

# Format lint (CI-style — no writes, exits non-zero on diff).
swift format lint --strict --recursive .

# Build & test individual packages (fast).
cd Packages/TransmissionRPC && swift test
cd Packages/TransmissionCore && swift test

# IMPORTANT: the default `swift` on this machine uses the Command Line Tools SDK,
# which does NOT ship Swift Testing's `Testing` module. Plain `swift test` fails
# with "no such module 'Testing'" even though the code is correct. Prefix every
# package-test invocation with the full Xcode toolchain:
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
cd Packages/TransmissionCore && swift test

# Build the macOS app target (slow; prefer the `xcode` MCP server's `BuildProject`).
xcodebuild -project TransmissionSwift.xcodeproj -scheme TransmissionSwift build | xcbeautify

# Run the macOS app's tests.
xcodebuild -project TransmissionSwift.xcodeproj -scheme TransmissionSwift test | xcbeautify

# Run all pre-commit hooks across the repo (uses prek).
prek run --all-files
```

When invoked from inside Xcode via Claude Code: prefer the `xcode` MCP server (`BuildProject`, `XcodeRefreshCodeIssuesInFile`, `RunSomeTests`) over raw `xcodebuild`. The MCP tools pre-parse output and save context.

## Conventions

- **Style:** enforced by `swift-format`. Config in `.swift-format`. Don't argue with it; run it.
- **Indentation:** 4 spaces (set by `.swift-format`).
- **Naming:** `PascalCase` for types, `camelCase` for properties/methods.
- **Types:** strong types, no force-unwrapping. Prefer typed errors over `Error` strings.
- **Comments:** rare. Only when *why* is non-obvious. No "what" comments next to self-explanatory code.
- **Tests:** Swift Testing framework (`@Test`, `#expect`). XCUIAutomation for UI tests.
- **Compiler strictness:** packages use `.swiftLanguageMode(.v6)` in `Package.swift` (Swift 6 mode = strict concurrency). `-warnings-as-errors` is applied by CI (`swift test -Xswiftc -warnings-as-errors`), **not** in `Package.swift` — it conflicts with Xcode's `-suppress-warnings` for package deps. Don't add it to the manifests.

## Architectural layering (compiler-enforced)

```
App target  ──depends on──>  TransmissionCore  ──depends on──>  TransmissionRPC  ──depends on──>  Foundation
```

If you need to add a dependency that crosses these boundaries the wrong direction, stop and propose a refactor instead.

## Local reference material & dev daemon

- `reference/` (gitignored) caches the upstream RPC specs — both the legacy protocol (4.0.6, **the one we implement**) and the JSON-RPC 2.0 protocol (4.1+). See `reference/README.md` to re-fetch.
- Local dev daemon: `transmission-daemon -g ~/.transmission-dev -t -u dev -v devpass -p 9091 -w /tmp/transmission-dev-downloads` (installed via `brew install transmission-cli`).
- RPC test fixtures in `Packages/TransmissionRPC/Tests/TransmissionRPCTests/Fixtures/` were captured from a real daemon with `curl` — recapture rather than hand-edit when the protocol surface grows.
- Opt-in E2E UI test (needs the daemon above): `TEST_RUNNER_TRANSMISSION_E2E=1 xcodebuild test -project TransmissionSwift.xcodeproj -scheme TransmissionSwift -only-testing:TransmissionSwiftUITests`.

## Working efficiently in this repo

- **Token economy** (sessions here default to a mid-tier model on purpose):
  - Delegate broad codebase exploration ("find where X is handled", "which views use Y") to a cheap subagent (Claude Code: the `Explore` agent or `Agent` tool with a `haiku` model) instead of reading many files in the main loop.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvacek/TransmissionSwift](https://github.com/jvacek/TransmissionSwift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
