---
trigger: always_on
description: Open-source, cross-platform SSH client with a single core. Kotlin Multiplatform, Compose
---

# Skerry

Open-source, cross-platform SSH client with a single core. Kotlin Multiplatform, Compose
Multiplatform UI, one codebase across Desktop (Linux, Windows, macOS) and Android at feature parity.
**iOS/iPadOS is deferred** — don't re-add its targets or `iosMain`.

## Commands

Requires **JDK 21** (`foojay-resolver` fetches one if needed) and an Android SDK for every client
build — `:androidApp` is always in the settings graph, so `ANDROID_HOME` (or `sdk.dir` in
`local.properties`) is needed even for a desktop-only build.

```bash
./gradlew :composeApp:run                                   # desktop
./gradlew :composeApp:packageDistributionForCurrentOS       # .deb / .rpm / .msi / .dmg
ANDROID_HOME=$HOME/Android/Sdk ./gradlew :androidApp:installDebug
./gradlew test allTests                                     # JUnit 5; `test` alone skips shared/composeApp
./gradlew :androidApp:compileDebugKotlin                    # Android side of a UI change
./gradlew build                                             # full gate, lint included
./gradlew detektAll                                         # static analysis; detektBaseline to re-baseline
./gradlew koverHtmlReport                                   # coverage report
docker compose up -d --build                                # sync server; set SKERRY_JWT_SECRET
./gradlew :server:run -PserverOnly                          # server-only build, no Android SDK
```

Test stack is `kotlin("test")` on the **JUnit 5** backend. There is no Kotest, no MockK, and no
detekt/ktlint in this repo — don't introduce them because a generic Kotlin skill suggests them.
Fakes are hand-written; the lint gate is Android lint inside `./gradlew build`.

## Repository layout

```
shared/       # KMP core: ssh/, sftp/, vault/, sync/, team/, share/, terminal/, ai/ (+ai/local),
              # telnet/, serial/, mosh/, rdp/, vnc/, graphics/, audio/, tunnel/, container/,
              # snippet/, runbook/, host/, tag/, files/, guard/, trust/, update/
              # commonMain + jvmSharedMain (shared JVM for desktop+Android) + desktopMain + androidMain
composeApp/   # UI (Compose Multiplatform): commonMain + androidMain + desktopMain
androidApp/   # Android app (MainActivity, manifest); applicationId app.skerry
server/       # self-hosted sync server (Ktor, AGPL-3.0)
sync-wire/    # wire contract shared by client and server (needed by server-only builds)
docs/         # HTML prototypes (source of truth for UX) and design documents
```

## How we work

Every change follows the same loop, but **what the loop demands depends on what the change is**.
Ask the harness rather than guessing:

```bash
tools/harness/gate.py status     # kind, areas, and everything still owed
tools/harness/gate.py task bug 133   # when the auto-detected kind is wrong
```

| Kind | How it is detected | What it owes |
|---|---|---|
| `docs` | no code in the diff | nothing — commit freely |
| `refactor` | `refactor/` `chore/` `perf/` branch | checks · tests · build · detekt · reviewers |
| `feature` | `feat/` branch, or an unnamed branch with code | the same, plus a test touched |
| `bug` | `fix/` `bug/` `hotfix/` branch, or declared | the same, plus a test recorded failing **before** the fix |

Areas add to that: UI or Android in the diff pulls in `:androidApp:compileDebugKotlin` and
`ecc:a11y-architect`; server pulls in `ecc:java-reviewer`; terminal pulls in
`ecc:performance-optimizer`. A declaration can make the gate stricter, never looser — a diff with
Kotlin in it is never treated as docs.

### 0. Orient before writing code

- **Read `docs/coding-guidelines.md`** — it encodes bugs we already paid for. Division of labour:
  this file owns the *process*, `coding-guidelines.md` owns *what the code must look like*
  (abstraction catalogue, decomposition, coroutine and security patterns, self-review checklist).
  A rule belongs in exactly one of the two.
- **Search for an existing abstraction before creating one** (guidelines §1). Second repetition is a
  signal, third makes extraction mandatory.
- For a non-trivial feature, map the ground first with `ecc:code-explorer` (how the existing
  subsystem works) and/or `ecc:code-architect` (where the new pieces belong). Skip for small fixes.
- Work happens on a feature branch. `main` is protected — every change lands through a PR.

### 1. RED — the failing test comes first

- Write the test before the implementation, in `commonMain` test sources unless the behaviour is
  genuinely platform-specific.
- Run it and **confirm it fails for the intended reason**, not on a compile error or a typo.
- For a bug fix the test must reproduce the bug, and the harness wants that on record:

  ```bash
  tools/harness/gate.py red --tests '*ReconcileDebt*' --file shared/src/commonTest/kotlin/.../ReconcileDebtStoreTest.kt
  ```

  It refuses to record a test that passes, and refuses a pattern that matched nothing. If the fix
  is already written, revert it, record RED, restore it — otherwise the bug fix cannot be committed.
- For controllers touching coroutines, cover cancellation and re-entry — that's the bug class
  guidelines §3 exists for.
- `ecc:tdd-guide` and the `ecc:kotlin-testing` skill are the reference for test shape; ignore their

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeCherkasov/SkerrySSH](https://github.com/SeCherkasov/SkerrySSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
