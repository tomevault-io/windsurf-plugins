---
trigger: always_on
description: handles, so do not "simplify" them away: stale `.gcda` **merge** into the next
---

# AGENTS.md — anoa

> This file is human-curated project knowledge for AI agents.
> Agents may propose updates, but humans approve them.
> Research shows human-written AGENTS.md improves agent success ~4%.

---

## Project Overview

- **Name**: anoa (binary: `anoa`)
- **Type**: native desktop/CLI binary — a browser automation host, not a Node service
- **Stack**: C++17 + Qt 6 (Widgets, WebEngineWidgets, WebEngineCore, Network, WebSockets), built with CMake ≥ 3.16. `find_package(Qt6 6.4 REQUIRED ...)` is the floor; CI pins 6.7.3. Node is used **only** for the test suites under `tests/`.
- **Description**: A Qt WebEngine browser that exposes the live page over an HTTP `/render/*` + screenshot API and an authenticated CDP WebSocket proxy, and can render that page into a terminal via `anoa terminal`.

---

## Conventions

### File Structure
```
src/
├── main.cpp        # entry point + the raw-argv pre-scan that selects the mode
├── browser/        # AnoaBrowser — the QWebEngineView host, profiles, extensions
├── cdp/            # CdpProxy (authenticated WebSocket proxy) + CDP extension methods
├── config/         # Config struct, parseArgs(), loadConfigFile() — shared by both modes
├── http/           # HttpServer — /json/*, /render/*, screenshot and input endpoints
├── pdf/            # PdfHandler — print-to-PDF
└── terminal/       # `anoa terminal`: viewer UI, HTTP transport, CDP transport
resources/          # .desktop file, SVG icon, anoa.sh bundle launcher
tests/
├── unit/           # QTest + CTest (test_config.cpp, its own CMakeLists.txt)
├── integration/    # vitest (*.test.js) + two bash suites (*.test.sh)
├── e2e/            # Playwright (TS) and Puppeteer (JS) against a running binary
└── regression/     # smoke.sh — fast post-commit check of the 5 critical paths
.github/
├── workflows/      # ci.yml, release.yml, update-homebrew-tap.yml
├── homebrew/       # anoa.rb.tpl (cask), anoa-linux.rb.tpl (formula)
└── entitlements/   # anoa.entitlements for macOS codesigning
```

### Naming Conventions
- Source files: `snake_case.cpp` / `snake_case.h`, one directory per subsystem
- Classes / structs: `PascalCase` — Qt classes keep their `Q` prefix
- Methods and locals: `camelCase`
- Member variables: `m_camelCase`
- Constants: `kCamelCase` for file-local `constexpr` values, `UPPER_SNAKE_CASE` for macros
- CLI flags: `kebab-case`; HTTP endpoints: `kebab-case` under `/render/*`, `/json/*`
- **String literals in `src/` must be ASCII-only** (this also builds on MSVC). Write
  UTF-8 bytes as escapes (`"\xE2\x80\x94"`). Non-ASCII in comments is fine.

### Code Patterns
- Every subsystem is a `QObject` owning its own Qt resources; parent-child ownership,
  not manual `delete`.
- Signals/slots across a seam, never a blocking call: `FrameBackend` requests a frame
  and is answered by `frameReady` / `frameFailed`, because a WebSocket transport
  cannot answer synchronously without a nested event loop.
- Config flows one way: `parseArgs()` → `Config` struct → constructors. Nothing
  re-reads argv.
- Validation failures in `parseArgs()` print one line to stderr and `::exit(1)`.
  Runtime failures inside terminal mode instead ask `exec()` to return non-zero so
  the termios/alt-screen restore still runs.
- `-Wall -Wextra -Wpedantic` is on for the main target and the tree is warning-clean;
  keep it that way.
- **Any POSIX call whose name collides with a `QObject` member must be `::`-qualified
  inside a `QObject` subclass** (`::connect`, `::bind`, `::listen`, `::accept`) —
  unqualified lookup finds `QObject::connect` and stops.

### Testing Conventions
- **Unit (QTest + CTest)**: `make test` — it re-configures with `-DBUILD_TESTS=ON`,
  builds and runs `ctest --output-on-failure`. A plain `make build` leaves
  `BUILD_TESTS` OFF, so the test target does not exist. Sources live in `tests/unit/`.
  `parseArgs()` cannot be tested in-process (it reads argv off the live
  `QCoreApplication` and exits on error), so those cases re-invoke the test binary
  through `QProcess` with `ANOA_TEST_HARNESS=parse_args` and parse a JSON line
  off stdout.
- **Integration (vitest)**: `cd tests/integration && npm install && npx vitest run`.
  `vitest.config.js` sets `fileParallelism: false` on purpose — each file spawns its
  own `anoa` on the same port triplet, so concurrent files would bind-clash
  or silently talk to the wrong instance. Binary and port come from `ANOA_BINARY` /
  `ANOA_PORT`.
- **Integration (bash)**: `bash tests/integration/port_layout.test.sh` and
  `bash tests/integration/extensions.test.sh`, both taking
  `ANOA_BINARY=./build/anoa`.
- **E2E**: `cd tests/e2e && npm install`, then `npx playwright test` (needs
  `npx playwright install --with-deps chromium`) and `node --test puppeteer.test.js`.
  Both attach to an already-running `anoa`; the test does not start it.
- **Regression**: `bash tests/regression/smoke.sh` — same `ANOA_BINARY`/`ANOA_PORT`
  contract.
- Tests run against a **real browser process**, never a mock. Each vitest and bash
  file starts and kills its own instance; the e2e suites are the exception and attach
  to one started outside them.
- Terminal-mode tests need a **pty**: `terminal_app.cpp` refuses to start unless both

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [porcupine-md/anoa-browser](https://github.com/porcupine-md/anoa-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
