---
trigger: always_on
description: - **Project knowledge goes in tracked Markdown, not auto-memory.** Decisions, conventions, gotchas, and status belong in checked-in `.md` files (`AGENTS.md`, `docs/`) where teammates can see them. Reserve per-user auto-memory for cross-project behavioral preferences.
---

## Operating principles

- **Project knowledge goes in tracked Markdown, not auto-memory.** Decisions, conventions, gotchas, and status belong in checked-in `.md` files (`AGENTS.md`, `docs/`) where teammates can see them. Reserve per-user auto-memory for cross-project behavioral preferences.
- **Verify before citing.** Don't claim a library API, version, flag, or service behavior from training data — check with context7 / WebFetch / WebSearch first, even for libraries you "know."
- **The human owns the code.** No `Co-Authored-By: Claude` / `🤖 Generated with Claude Code` trailers on commits or PRs. To disclose AI assistance (optional), use a single `Assisted-by: Claude <model-id>` trailer. AI-assisted PRs are reviewed like any other; the author is accountable.

## Project

Compose for Desktop application that emulates OCPP charge points for testing CSMS backends. Kotlin Multiplatform with a single `jvm()` target; requires JDK 25 (`jvmToolchain(25)`).

There are two Gradle modules (settings.gradle.kts): `:engine` — the headless emulator (OCPP 1.6 protocol, charge-point domain, platform infrastructure, database) with no Compose dependency; and `:app` — the Compose desktop UI (window/navigation, self-updater) which depends on `:engine`. This started as a single `:app` module (itself a merge of the older `:common` + `:v16`), and `:engine` was extracted so the emulation logic can be driven without a UI. The README mentions OCPP 2.0.1 / `v201`, but that module does not exist in this repo yet.

The `com.monta.ocpp.emulator.v16` **package** (and the `ocpp-v16` library dependency) keep the `v16` name — that's the OCPP 1.6 protocol version, not a module coordinate. Only the module/directory is `app`.

## Commands

```shell
./gradlew :app:run                       # run the emulator app
./gradlew :engine:jvmTest                # run tests (the test suite lives in :engine)
./gradlew :engine:jvmTest --tests "com.monta.ocpp.emulator.platform.util.PrettyJsonFormatterTest"   # single test class
./gradlew ktlintCheck                    # lint
./gradlew ktlintFormat                   # auto-format
./gradlew :app:packageDistributionForCurrentOS   # build native installer (Dmg/Deb/Rpm/Exe)
```

CI (`.github/workflows/pull_request.yml`) runs `:engine:test` (an alias for `jvmTest`) with Kover coverage, plus detekt via `monta-app/detekt-action`. All tests currently live in `:engine`; `:app` has no test sources of its own. Tests use JUnit 5 / kotlin-test.

## Code style

ktlint (`intellij_idea` style, trailing commas required on both call and declaration sites — see `.editorconfig`) covers formatting. IntelliJ run configs live in `.run/`.

Beyond what ktlint checks, this codebase prefers **explicit over terse**. A reader scanning a call site should never have to infer a receiver's type, an implicit parameter's meaning, or where a branch ends. These four rules are **not machine-enforced** (see below) — they are review rules, so check them yourself before you call a change done:

1. **Block bodies, not expression bodies.** Always `fun f(): T { return x }`, never `fun f() = x`. Applies to functions and to DAO→DTO mappers alike.
2. **Braces on every `if`.** Never a braceless single-line body, not even for a bare `return` or `throw`. This extends to `if`/`else` used as an *expression* — brace both branches instead of `if (x) A else B`.
3. **Name every lambda parameter; never use `it`.** `.map { connector -> connector.position }`, not `.map { it.position }`. Kotest's `it("describes a case")` is the spec DSL, not a lambda parameter — leave those alone.
4. **No chained elvis fallbacks.** A single `?:` supplying a default on a nullable is fine (`connector?.kw ?: 0.0`); chaining `?:` as a lookup cascade is not. Prefer a named helper with an early `return`, or `requireNotNull`/`checkNotNull` with a message, over `a ?: b ?: error(...)`.

## Architecture

### Package layout

Both modules use the same `com/monta/ocpp/emulator/` package root, split by concern: the headless domain/protocol/platform code lives in `engine/src/jvmMain/...`, the Compose UI in `app/src/jvmMain/...`. The role folders below apply the same way in whichever module a package lives (e.g. `chargepoint/core/entity` + `repository/` + `service/` are in `:engine`, while `chargepoint/core/ui/` is in `:app`).

```
:app root:    App.kt MainWindow.kt AppKoinModule.kt   entry point, window registration, app DI module
:engine root: EngineKoinModule.kt                     engine DI module (@ComponentScan, OCPP client)
chargepoint/        the domain aggregate — charge point → connector → transaction
  core/             the aggregate root's own layers — model/ entity/ repository/ service/ exception/
                    ui/ (grouped by screen: list/ detail/ form/ pbm/ security/ + shared component/)
  connector/        entity/ model/ repository/ service/ ui/
  transaction/      entity/ repository/ service/
  txdefault/        entity/ repository/ service/
vehicle/            model/ service/ ui/
interceptor/        the 🤓 message-interception feature — model/ service/ ui/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monta-app/ocpp-emulator](https://github.com/monta-app/ocpp-emulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
