---
trigger: always_on
description: Do not add Co-Authored-By lines to git commits.
---

# Shellify — Claude Code Guide

Do not add Co-Authored-By lines to git commits.

**README rule:** Before working in any directory, read its README.md if one exists. When your changes affect a directory, update its README.md to reflect them. When creating a new directory, create a README.md.

Shellify. Wraps websites in isolated WebView containers with per-app ad blocking, biometric lock, and encrypted backup. Local-first, no cloud, no analytics.

---

## Build Commands

```bash
./gradlew assembleDebug          # build debug APK
./gradlew installDebug           # install on connected device
./gradlew testDebugUnitTest      # unit tests (includes Konsist arch checks)
./gradlew verifyRoborazziDebug   # screenshot regression tests
./gradlew recordRoborazziDebug   # regenerate screenshot goldens (after UI changes)
./gradlew detekt                 # static analysis
./gradlew lintDebug              # lint
./gradlew detekt lintDebug testDebugUnitTest  # full local check suite
```

---

## Quality Checks

After making any code change, run all applicable checks. **Always launch them as parallel Bash tool calls in a single message — never chain them with `&&` or run them sequentially.**

| Check | Command | When to run |
|---|---|---|
| Static analysis | `./gradlew detekt --continue` | Every change |
| Lint | `./gradlew lintDebug --continue` | Every change |
| Unit tests + arch | `./gradlew testDebugUnitTest --continue` | Every change |
| Screenshot regression | `./gradlew verifyRoborazziDebug --continue` | UI changes only |

**Parallel example** — fire all four in one message with separate Bash tool calls:
- Bash: `./gradlew detekt --continue`
- Bash: `./gradlew lintDebug --continue`
- Bash: `./gradlew testDebugUnitTest --continue`
- Bash: `./gradlew verifyRoborazziDebug --continue`  ← skip if no UI touched

If screenshot tests fail after a UI change, regenerate goldens (`./gradlew recordRoborazziDebug --continue`) and commit the updated images alongside the code change.

---

## Architecture

Clean Architecture with strict layer separation enforced at compile time by **Konsist** (`app/src/test/java/io/shellify/app/konsist/ArchitectureTest.kt`).

```
feature:*  →  core:*  →  core:domain
app                       (NavHost, DI wiring, ShellifyApplication)
```

- `feature:*` — Compose screens + ViewModels. May depend on `core:*` infrastructure but never on `core:database` or other `feature:*` modules.
- `core:*` — infrastructure implementations. May depend on `core:domain`; must not depend on `presentation` or `data`.
- `core:domain` — pure Kotlin: models, repository interfaces, use cases. Zero Android deps.

**Hard rules enforced by Konsist (build fails if violated):**
- `feature:*` must not import `core:database` directly
- `feature:*` must not import other `feature:*` modules
- `core:domain` must not have any Android dependencies
- ViewModels must not import `domain.repository` interfaces directly — use use cases instead
- `*UiState` classes must be `data class` and live in `presentation..*`
- `*ViewModel` classes must extend `androidx.lifecycle.ViewModel` and live in `presentation..*`
- Use cases must only import from the domain layer

**Known gap — Konsist does not check Activities.** Activities may import `core.*` infrastructure directly when they own system UI integration (window insets, biometrics, engine lifecycle) that no ViewModel can hold. Keep Activities thin: no business logic, no state mutation — delegate everything to the ViewModel.

**Known compromise — ViewModels may use `core.*` infrastructure directly** (e.g. `IsolationManager`, `PasswordManager`) when no suitable use case exists. Konsist only blocks `domain.repository` imports. Prefer wrapping in a use case for new work.

DI is **manual** — no Hilt/Koin. Dependencies are wired in `ShellifyApplication` and passed down via ViewModel factories (`ViewModelProvider.Factory` inner class on each ViewModel).

### MVVM pattern (all feature modules)

Every screen follows the same structure:

| Piece | Rule |
|---|---|
| `*ViewModel` | Holds all state + business logic. Uses `viewModelScope`, exposes `StateFlow<*UiState>`. One-shot events (navigation, toasts) via `SharedFlow<*Command>`. |
| `*UiState` | Immutable `data class`. Single source of truth the screen observes. |
| `*Command` | Optional `sealed interface` for one-shot side-effects the View must execute (e.g. `NavigateTo`, `Finish`, `LoadUrl`). |
| Screen / Activity | Pure View layer. Observes `uiState`, collects commands, forwards gestures to ViewModel. No coroutine scope of its own — use `lifecycleScope`. No direct data or infrastructure calls. |

`feature/webview` is the exception: it uses `FragmentActivity` (not a Compose screen) because it owns a raw WebView/GeckoView hierarchy and system UI that cannot live in a ViewModel. The same ViewModel + UiState + Command pattern still applies.

---

## Module Layout

```
app/           → ShellifyApplication, MainActivity, NavHost, DI wiring
build-logic/   → Gradle convention plugins (no boilerplate in modules)
core/domain    → Models, repository interfaces, use cases (pure Kotlin)
core/database  → Room + SQLCipher, DAOs, entities, migrations
core/crypto    → AES-256-GCM, Argon2id, PBKDF2
core/security  → Android Keystore, biometrics, password management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smellouk/shellify](https://github.com/smellouk/shellify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
