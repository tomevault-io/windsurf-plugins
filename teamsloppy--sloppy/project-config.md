---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.
Project type: Swift Package (Swift 6.x) + React/Vite dashboard.

## Scope and stack

- Package manager: SwiftPM (`Package.swift`)
- Swift platform: macOS 14+
- Executables: `sloppy`, `SloppyNode`
- Libraries: `Protocols`, `PluginSDK`, `AgentRuntime`
- Dashboard: `Dashboard/` (`react`, `vite`)
- Persistence: SQLite (`sqlite3`)
- Test framework: Swift Testing (`import Testing`, `@Test`, `#expect`)

## Build, lint, test, run

Run from repo root unless noted.

### Resolve dependencies

- `swift package resolve`

### Build (Swift)

- `swift build`
- `swift build -c release`
- `swift build -c release --product sloppy`
- `swift build -c release --product SloppyNode`

### Test (Swift)

- Full suite: `swift test`
- Parallel suite: `swift test --parallel`
- List tests: `swift test list`

### Run a single Swift test (important)

- By exact name:
  - `swift test --filter CoreTests.postChannelMessageEndpoint`
- By test group:
  - `swift test --filter CoreTests`
  - `swift test --filter AgentRuntimeTests`
- Note: `swift test --list-tests` is deprecated; use `swift test list`.

### Run executables

- `swift run sloppy`
- `swift run SloppyNode`

Useful `sloppy` flags:

- `swift run sloppy --oneshot`
- `swift run sloppy --run-demo-request`
- `swift run sloppy --config-path sloppy.json`

### Dashboard commands (inside `Dashboard/`)

- `npm install`
- `npm run dev`
- `npm run build`
- `npm run preview`

### Lint/format status

No dedicated lint/format config is committed for SwiftLint, swift-format, ESLint, or Prettier.
When changing code, preserve local style and validate with:

- `swift test --parallel`
- `swift build -c release --product sloppy`
- `npm run build` (when dashboard files change)

## CI parity checklist

CI (`.github/workflows/ci.yml`) runs:

- `swift test --parallel`
- `swift build -c release --product sloppy`
- `swift build -c release --product SloppyNode`
- `npm install` + `npm run build` in `Dashboard/`
Keep local changes green for the same command set.

## Releases and install from GitHub

Pushing a tag `v*` runs [`.github/workflows/release.yml`](.github/workflows/release.yml): Swift tests and release builds on Linux and macOS, Dashboard `npm run build` on both, tarballs uploaded to the GitHub Release together with `SHA256SUMS.txt` and `sloppy-version.json`, release notes from **GitHub auto-generated changelog** (`generate_release_notes: true`), prerelease when the version segment looks like alpha/beta/rc/preview/pre. The workflow then commits updated [`Casks/sloppy.rb`](Casks/sloppy.rb) (macOS) and [`Formula/sloppy.rb`](Formula/sloppy.rb) (Linuxbrew) on the repository default branch.

Install prebuilt binaries without building: [`scripts/install.sh`](scripts/install.sh) `--release` (uses `SHA256SUMS.txt` from the release). Set `SLOPPY_RELEASE_REPO`, `SLOPPY_RELEASE_TAG`, or `SLOPPY_LOCAL_ROOT` as needed.

## Code style guide

### Swift: imports and formatting

- Use 4-space indentation and no tabs.
- Keep imports minimal; place `Foundation` first when used.
- Follow existing multiline style and trailing commas.
- Keep files focused; extract helpers instead of large monolith methods.

### Swift: types and naming

- `UpperCamelCase` for types; `lowerCamelCase` for vars/functions.
- Prefer `struct` for DTO/protocol models; use `actor` for shared mutable state.
- Mark cross-target API explicitly with `public`.
- Add `Sendable` where values cross concurrency boundaries.
- Use enums for constrained state/actions (`RouteAction`, `WorkerMode`, etc.).
- For API-compatible values, use explicit raw values (snake_case when needed).

### Swift: error handling and resilience

- Use `throws` for recoverable boundary failures.
- In router/http boundaries, convert invalid payloads to stable 4xx responses.
- Prefer graceful fallback over crashing in runtime services.
- Avoid force unwraps and `fatalError` in production paths.
- Log operational failures with context and continue when safe.

### Swift: concurrency and architecture

- Prefer actor isolation to locking.
- Keep orchestration async end-to-end (`async`/`await`).
- Do not bypass actor boundaries with shared mutable globals.
- Maintain separation: transport (`CoreHTTPServer`) -> routing (`CoreRouter`) -> service (`CoreService`) -> runtime (`AgentRuntime`) -> persistence (`SQLiteStore`).

### Tests

- Use Swift Testing macros (`@Test`, `#expect`).
- Write behavior-focused tests with clear arrange/act/assert flow.
- Keep tests deterministic and isolated.
- For endpoint logic, test via router/service with realistic payloads.

### Dashboard (React)

- Use function components and hooks.
- Use named exports for components/utilities.
- Keep state local; derive computed values with `useMemo` when useful.
- Use `async/await` and handle non-OK responses explicitly.
- Match existing JS formatting: 2-space indent, semicolons, double quotes.
- For dropdown/select UI, always use the custom `.actor-team-search` dropdown pattern (see `ActorsView.tsx` and `actors.css`) — never use native `<select>` elements.

## Module map

### Swift targets

- `Sources/Protocols`
  - Shared domain and wire models (`APIModels`, `RuntimeModels`, JSON helpers, envelopes).
  - Base dependency for all runtime-facing modules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TeamSloppy/Sloppy](https://github.com/TeamSloppy/Sloppy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
