---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Softcover is a Kotlin Multiplatform / Compose Multiplatform client for [Hardcover.app](https://hardcover.app/), a book tracking platform. It ships on Android (SDK 26+), iOS, and desktop (JVM), with shared UI and logic in `commonMain` and thin platform seams.

## Engineering principles

**Never take shortcuts; never propose the "less clean" option.** When two solutions are available — a structurally clean one and a smaller-diff pragmatic one — pick the clean one and present it as the recommendation. Do not surface "less clean / pragmatic / repository-aggregator / pass-through delegation / cross-feature data-source reach" alternatives as primary recommendations. Mention a smaller-diff fallback only when the user explicitly asks for the cheaper path or when the clean option is genuinely out of scope. Cost (larger diff, more files touched, follow-up moves) is not a reason to defer; surface the cost transparently and proceed with the right structure unless told otherwise.

## Build & Test Commands

```bash
./gradlew assembleDebug          # Debug build
./gradlew assembleRelease        # Release build
./gradlew test                   # Run unit tests
./gradlew :app:test              # Run unit tests for app module
./gradlew connectedAndroidTest   # Run instrumented tests (requires device/emulator)
./gradlew lint                   # Run Android Lint
```

The project uses `kotlin.code.style=official`. Both the foundation ktlint ruleset and detekt **are**
configured and gated (see Code Style below); `./gradlew styleCheck` runs detekt + the mechanical checks,
and `./gradlew check` runs the full set.

## Design System

The brand-agnostic design skeleton (theme/typography plumbing, layout primitives, the shared component catalog, the editorial role contract) is governed by the foundation [`docs/rhaydus/0.3.1/design-system-foundations.md`](docs/rhaydus/0.3.1/design-system-foundations.md). [docs/reference/design-system.md](docs/reference/design-system.md) is the source of truth for Softcover's brand layered on top — color roles, editorial typography values, brand components, patterns, decision rules. It is split into section files under [`design-system/`](docs/reference/design-system/) behind a thin index; **read only the section you need** rather than the whole doc. Consult both before designing or modifying any UI surface.

**Maintenance rule (enforced by review).** Any change that introduces, retires, or alters a foundation, component, or pattern in the design system MUST update the relevant section file under `docs/reference/design-system/` in the same change (the doc is split into section files behind the thin `docs/reference/design-system.md` index). The `rhaydus-kotlin:code-reviewer` agent treats a design-system change without a corresponding doc update as a blocker. Examples that require a doc update: a new shared component under `core/presentation/component/`, a new editorial typography role, a new color role usage, a new layout pattern that other screens should adopt, retirement or renaming of any of the above. Localized tweaks to a single screen that don't change the system itself do not require an update.

## Code Style

The shared Kotlin code style is governed by the foundation [`docs/rhaydus/0.3.1/code-style.md`](docs/rhaydus/0.3.1/code-style.md) — the source of truth for naming, layout, and whitespace. [docs/reference/code-style.md](docs/reference/code-style.md) keeps only Softcover-specific deltas (the Apollo/AppLog error-handling bindings). Read both before writing or modifying Kotlin code.

The mechanical style rules are enforced by tooling, not manual vigilance — for every developer, with zero setup, via the Gradle `check` lifecycle (so CI gates on them too):

- **The foundation ktlint ruleset** (`nl.rhaydus:ktlint-rules`) **auto-fixes and gates** the mechanizable layout rules. Run `./gradlew ktlintFormat` to auto-fix, `./gradlew ktlintCheck` to gate (also run by `check`). The rules: multi-arg one-per-line wrapping (2+ args/params, even when they fit — exempting collection factories, `Modifier.…` chains, trailing-lambda calls), trailing comma on multi-line lists, blank line after `super.*()` / `AppLog.e(...)`, `// region`/`// endregion` flush, no blank line after `{` / before `}`, blank line between sibling composables, and boolean `!` → `.not()` (gate-only; fix by hand).
- **Five formerly-greppable rules are now blocking ktlint rules** in `nl.rhaydus:ktlint-rules` (gate-only, fixed by hand; gated by `ktlintCheck`): inline fully-qualified references, one-type-per-file, project-import ordering, inline mockk stubs (`coEvery`/`every` one-liners open onto their own line), and bare `runCatching` in a use case (use `runCatchingLogged`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CinqueIzumi/Softcover](https://github.com/CinqueIzumi/Softcover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
