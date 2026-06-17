---
trigger: always_on
description: This file is read automatically at the start of every Claude Code session in this repository.
---

# CLAUDE.md — instructions for Claude Code

This file is read automatically at the start of every Claude Code session in this repository.

---

## 1. MANDATORY SESSION STARTUP — read these first

In order, before responding to any user request:

1. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/MEMORY.md` → index of memory files
2. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/session_handoff.md` → what happened last (when present)
3. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/project_status.md` → roadmap progress (when present)
4. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/feedback_working_style.md` → how this user works
5. `PRD.md` → product requirements
6. `ROADMAP.md` → current phase + acceptance criteria
7. `LESSONS.md` → numbered lessons (never repeat past mistakes)

---

## 2. PROJECT IDENTITY

**What:** Native iOS + watchOS personal scheduling app with military-precision time blocks. Single-user. Reliable medication reminders. Holistic vacation module. CloudKit sync.

**What it is NOT:** not multi-user, not a calendar replacement, not a habit gamifier, not analytics-heavy, not for sale (yet), no backend.

**Privacy stance:** All data stays in the user's Apple ecosystem. No telemetry. No third-party SDKs.

**Languages:** UI in ES + EN + FR. Documentation in English. Conversation in user's preferred language (Spanish currently).

---

## 3. TECH STACK (verified 2026-04-25)

| Layer | Choice | Version |
|---|---|---|
| Language | Swift | 6.0+ |
| UI | SwiftUI | iOS 18+, watchOS 11+ |
| Persistence | SwiftData (preferred) or CoreData + CloudKit | iOS 18+ |
| Sync | CloudKit Private Database | — |
| Health | HealthKit (Medications + Sleep) | iOS 18+ |
| Calendar | EventKit | — |
| Contacts | Contacts framework | — |
| OCR | VisionKit + Vision | — |
| AI | Apple Intelligence Foundation Models | iOS 18.1+ |
| Weather | WeatherKit | — |
| Marine | Open-Meteo Marine API | — |
| Currency | Frankfurter API | — |
| CI | GitHub Actions (`macos-latest`) | — |
| Lint | SwiftLint + swift-format | latest |

---

## 4. ARCHITECTURE PATTERNS

- **One type per file.** File name = type name.
- **MVVM with SwiftUI.** Views observe `@Observable` view models. No business logic in views.
- **Shared module** for code used by both iOS + watchOS targets (in `App/Shared/`).
- **Feature-folder layout** (`App/PersonalHygiene/Features/<Module>/`), not type-folder layout.
- **Async/await first.** No completion handlers in new code.
- **No third-party dependencies** unless absolutely necessary — prefer Apple frameworks.
- **i18n: every UI string** goes through `Localizable.xcstrings`. Hardcoded user-facing strings = build error (SwiftLint custom rule TBD).

---

## 5. TECHNICAL LESSONS — quick reference

See [LESSONS.md](LESSONS.md) for full text. Quick table:

| L0NN | Rule | Guard |
|---|---|---|
| L001 | Keep `ModelContainer` alive for the lifetime of any `ModelContext` built from it; never drop it from a helper | `SwiftDataRoutineRepositoryTests` exercises insert + cascade-delete + relationship-append (suite crashes if regressed) |
| L002 | A notification-identifier parser must recognize every module's prefix; model the prefix as an enum so adding a kind without updating the parser is a compile error | `BlockNotificationIdentifierTests.test_parse_recognizesAllKnownPrefixes` iterates `NotificationKind.allCases` and round-trips each |
| L003 | Files in `App/Shared/` using iOS-only APIs (UIGraphicsPDFRenderer, UIActivityViewController, etc.) must be `#if canImport(UIKit) && !os(watchOS)`-guarded — the watch target compiles `Shared/` too | `./scripts/deploy-watch.sh --no-install` builds the watch scheme; run after touching `Shared/` |
| L004 | Tab-root views inside iOS 18 TabView "More" overflow must NOT add their own `NavigationStack` (More provides one); doing so produces two stacked back chevrons on every push | Manual on-device check: open any overflowed tab → push into a child → exactly one back arrow |
| L005 | Test process crashes (signal-trap / "Restarting after unexpected exit") must NOT be filtered as the LLDB glitch in `check-tests.sh` — a real crash with no failed-test-method line silently passed CI | `scripts/check-tests.sh` counts process-crash lines separately; treats them as real failures regardless of exit code |
| L006 | `Text(LocalizedStringKey("foo.\(rawValue)"))` looks up `"foo.%@"`, not the runtime key — UI shows raw keys (`category.work` etc.). Use `Text(localizedKey: "foo.\(rawValue)")` for discrete-suffix lookups; xcstrings keys with placeholders must include the matching `%@` / `%lld` suffix | `BundleLocalizationLookupTests` resolves every enum-driven discrete-suffix key + every format-string key through `Bundle.main`; round-19 SwiftLint custom rule `dynamic_localized_key` rejects new `LocalizedStringKey("...\(...)")` at compile time; backup `scripts/check-localized-key-usage.py` and `scripts/check-xcstrings-format-consistency.py` static scans run alongside |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tandori46001/personal-hygiene](https://github.com/tandori46001/personal-hygiene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
