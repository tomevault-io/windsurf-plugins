---
trigger: always_on
description: - `Sources/CodexBar`: Swift 6 menu bar app (usage/credits probes, icon renderer, settings). Keep changes small and reuse existing helpers.
---

# Repository Guidelines

## Project Structure & Modules
- `Sources/CodexBar`: Swift 6 menu bar app (usage/credits probes, icon renderer, settings). Keep changes small and reuse existing helpers.
- `Sources/AgentMicro`: AgentMicro's focused menu bar executable. Keep the V1 surface Codex-only and read-only.
- `Tests/CodexBarTests`: XCTest coverage for usage parsing, status probes, icon patterns; mirror new logic with focused tests.
- `Tests/AgentMicroTests`: focused AgentMicro model and state-reducer tests. Prefer these seams over live AppKit tests.
- `docs/agentmicro`: AgentMicro product definition, V1 specification, roadmap, implementation log, and backlog.
- `Scripts`: build/package helpers (`package_app.sh`, `sign-and-notarize.sh`, `make_appcast.sh`, `build_icon.sh`, `compile_and_run.sh`). Release wrappers call `Scripts/mac-release`, which resolves `MAC_RELEASE_TOOL` or the shared `agent-scripts` checkout.
- `docs`: release notes and process (`docs/RELEASING.md`, screenshots). Root-level zips/appcast are generated artifacts—avoid editing except during releases.

## Build, Test, Run
- AgentMicro dev run: `AGENTMICRO_BUILD_ONLY=1 swift run --disable-automatic-resolution AgentMicro`.
- AgentMicro focused tests: `AGENTMICRO_BUILD_ONLY=1 swift test --disable-automatic-resolution --filter AgentMicro`.
- `AGENTMICRO_BUILD_ONLY=1` excludes the upstream CodexBar app targets and Sparkle artifact from the build graph; pair it with `--disable-automatic-resolution` so the upstream `Package.resolved` remains unchanged.
- Dev loop: `./Scripts/compile_and_run.sh` kills old instances, builds, packages, relaunches `CodexBar.app`, and confirms it stays running; add `--test` for the sharded full suite.
- Quick build/test: `swift build` (debug) or `swift build -c release`; `make test` for the sharded full suite.
- Package locally: `./Scripts/package_app.sh` to refresh `CodexBar.app`, then restart with `pkill -x CodexBar || pkill -f CodexBar.app || true; cd /Users/steipete/Projects/codexbar && open -n /Users/steipete/Projects/codexbar/CodexBar.app`.
- Release flow: `./Scripts/release.sh`; app metadata lives in `.mac-release.env`, repo build/signing stays in `Scripts/sign-and-notarize.sh`, and validation steps live in `docs/RELEASING.md`.

## Coding Style & Naming
- Enforce SwiftFormat/SwiftLint: run `swiftformat Sources Tests` and `swiftlint --strict`. 4-space indent, 120-char lines, explicit `self` is intentional—do not remove.
- Favor small, typed structs/enums; maintain existing `MARK` organization. Use descriptive symbols; match current commit tone.

## Testing Guidelines
- Add/extend XCTest cases under `Tests/CodexBarTests/*Tests.swift` (`FeatureNameTests` with `test_caseDescription` methods).
- Add AgentMicro-specific coverage under `Tests/AgentMicroTests`.
- Swift Testing: prefer backticked sentence names; no camelCase.
- Model names in tests/code: released models or clearly fictitious names only; never expose unreleased names.
- Always run `make test` before handoff; add focused `swift test --filter ...` runs for parser/provider fixes when possible.
- After any code change, run `make check` and fix all reported format/lint issues before handoff.
- Prefer CLI/focused tests over app-bundle live tests when behavior can be verified without relaunching CodexBar.
- Never run tests/checks or ad-hoc validation that can display macOS Keychain prompts. Live provider probes, browser-cookie imports, `codexbar usage` against real accounts, and real SecItem reads must be explicitly requested; otherwise use parser tests, stubs, test stores, or `KeychainNoUIQuery`.
- macOS CI is brittle around headless AppKit status/menu tests. Prefer covering menu behavior through stable state/model seams (`MenuDescriptor`, `ProvidersPane`, `CodexAccountsSectionState`, etc.) instead of constructing live `NSStatusBar`/`NSMenu` flows unless the AppKit wiring itself is the thing under test.

## Commit & PR Guidelines
- Commit messages: short imperative clauses (e.g., “Improve usage probe”, “Fix icon dimming”); keep commits scoped.
- PRs/patches should list summary, commands run, screenshots/GIFs for UI changes, and linked issue/reference when relevant.

## AgentMicro Product Documentation
- `docs/agentmicro/README.md` is the index for AgentMicro product truth. Read the relevant product docs before changing scope, task-state semantics, menu behavior, privacy boundaries, or roadmap commitments.
- Update `docs/agentmicro/V1_SPEC.md` when V1 behavior or acceptance criteria change, and update `docs/agentmicro/PRODUCT.md` when the product definition or long-term boundary changes.
- Record every completed product, architecture, implementation, documentation, or release change in `docs/agentmicro/PROJECT_LOG.md`. Include the date, impact, verification, and known limitations. Do not use the upstream root `CHANGELOG.md` for AgentMicro work.
- Put new ideas and deferred requirements in `docs/agentmicro/BACKLOG.md` before expanding the active version. Promote work into `docs/agentmicro/ROADMAP.md` only when its milestone and entry conditions are explicit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fizzy718/AgentMicro](https://github.com/fizzy718/AgentMicro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
