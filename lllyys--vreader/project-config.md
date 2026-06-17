---
trigger: always_on
description: Shared instructions for all AI agents (Claude, Codex, etc.).
---

# [AGENTS.md](http://AGENTS.md)

Shared instructions for all AI agents (Claude, Codex, etc.).

## Platforms (iOS + Android)

vreader is a **native iOS app** (Swift, at the repo root: `vreader/` +
`*.xcodeproj`) and — as of the ADR-0001 Android port — a **native Android
app** (Kotlin + Compose, under `android/`). They are two independently
shippable apps sharing identity/library/backup contracts, NOT a
cross-platform rewrite. **Source of truth for the Android strategy:
`docs/decisions/0001-android-port-strategy.md`.**

- **Path ownership** — iOS code = `vreader/`, `vreaderTests/`,
  `*.xcodeproj`, `project.yml`; Android code = `android/`, `spikes/`,
  `buildSrc/`, `gradle/`, root Gradle files, `gradlew*`, `gradle.properties`,
  `*.kt[s]`, `AndroidManifest.xml`, any `res/` tree; shared = `docs/`,
  `contracts/`, `dev-docs/`, `.claude/`, this file. (The audit gate's
  code-vs-docs classifier `.claude/hooks/lib/code-paths.sh` decides which
  PRs need a Codex audit — it covers the Android/Kotlin/`contracts/` code
  paths but is a boolean gate, not a full ownership taxonomy; `project.yml`
  / `*.xcodeproj` are owned-by-iOS here but ride along with `vreader/`
  changes.) Write isolation is binding — see
  `.claude/rules/48-parallel-execution.md` ("Cross-platform write isolation").
- **Tests**: iOS via `scripts/run-tests.sh` (xcodebuild). Android via
  Gradle (`./gradlew test` / `connectedAndroidTest`) — *the Android module
  lands in Phase 2 (#106); not wired yet.*
- **Release semantics**: per-platform version files + tags (iOS plain
  `vX.Y.Z`, Android `android/vX.Y.Z`) — see
  `.claude/rules/40-version-bump.md` ("Multi-platform").
- The Android port runs through the same 6-gate feature workflow; Phase 0
  (#103) path-scopes the automation so an `android/` PR is gated, not
  bypassed.

- You are an AI assistant working on the project.
- **Read `docs/architecture.md` before making any code changes. Update it when adding new layers, patterns, services, or changing how components communicate.**
- Use English unless another language is requested.
- Follow the working agreement:
  - Run `git status -sb` at session start.
  - Read relevant files before editing.
  - Keep diffs focused; avoid drive-by refactors.
  - Do not commit unless explicitly requested.
  - Keep code files under \~300 lines (split proactively).
  - Keep features local; avoid cross-feature imports unless truly shared.
  - **Research before building**: For new features, search for industry best practices,
    established conventions, and proven solutions (web search, official docs, prior art in
    popular open-source projects). Don't invent when a well-tested pattern exists.
  - **Edge cases are not optional**: Brainstorm as many edge cases as possible — empty input,
    null/undefined, max values, concurrent access, Unicode/CJK, RTL text, rapid repeated
    actions, network failures, permission denials. Write tests for every one.
  - **Test-first is mandatory** for new behavior:
    - Write a failing test (RED), implement minimally (GREEN), refactor (REFACTOR).
    - Coverage thresholds are enforced — `ut` fails if coverage drops.
    - Exceptions: CSS-only, docs, config. See `.claude/rules/10-tdd.md` for full scope.
  - Run unit-test gates through **`scripts/run-tests.sh`** (wraps `xcodebuild test -only-testing:vreaderTests` with a hard wall-clock watchdog, exact-pid wait, and an unambiguous `RUN-TESTS RESULT:` line). Skip UI tests during development. **Never drive the simulator (`sim-tap` / `idb` / `simctl openurl eval` / screenshots / verification) while a test run is in flight against the same UDID — sim contention wedges `xcodebuild test` into a 0%-CPU, zero-output ghost that lingers for hours.** Binding details + the recurring-incident write-up in `.claude/rules/52-test-sim-isolation.md`.
  - Default simulator: **iPhone 17 Pro** (Dynamic Island — catches safe area bugs).
  - **Test-book fixtures**: real EPUB/TXT/AZW3 books for manual/device testing live in `test-books/books/` (gitignored, local-only — not in CI), organized into `azw3/`, `epub/`, `txt/` subdirs and spanning English + large-CJK content and Kindle/KF8. Browse the directory for the current set; import to the simulator via the `sim-transfer` skill.
  - **Real books first** (binding): any test or verification that needs a book uses a real book from `test-books/books/` **first**. Construct a synthetic fixture (DebugBridge `DebugFixtureCatalog` entry, hand-built EPUB/TXT) **only** when no real book satisfies the condition — legitimate cases are: the format has no real book (no real PDF or MD today), the test needs a deterministic tiny structure a 13–18M book can't give cheaply (exact chapter count, RTL, controlled char offsets), or it's a CI unit test (which can't read the gitignored `test-books/`). State which exception applies when choosing synthetic. Synthetic fixtures pass while real-world structure / CJK pagination / large-file performance / AZW3-KF8 quirks stay unverified — so they're the fallback, not the default.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lllyys/vreader](https://github.com/lllyys/vreader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
