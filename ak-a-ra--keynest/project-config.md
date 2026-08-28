---
trigger: always_on
description: KeyNest: Ultra-fast, encrypted API Key & Developer Secret Vault for Android.
---

# AGENTS.md

## Project

KeyNest: Ultra-fast, encrypted API Key & Developer Secret Vault for Android.
Single-activity Kotlin/Compose Android app.

- **Tech Stack:** Kotlin, Jetpack Compose, Android (Material Design 3)
- **Build Command:** `./gradlew assembleDebug` or `compile_applet`
- **Test Command:** `./gradlew testDebugUnitTest` (CI verify gate runs this; `./gradlew test` also covers release-variant tests)

## Code philosophy — ponytail (YAGNI-first)

Before adding code, in order: does this need to exist? → already in the codebase? → stdlib? → native platform feature? → an already-installed dependency? → a one-liner? → only then write new code. Don't add abstractions with one implementation, config nobody sets, or a layer with one caller. Never skip validation, error handling, or a security measure to hit this bar — "minimal" means fewest moving parts, not fewer safeguards.

## Operating Modes (Active until user says "exit mode")

- **Conversation:** Always activate Caveman skill in fullmode.
- **Coding:** Always follow Ponytail methodology (minimalist, YAGNI-first).

## 🔨 Tool Usage Rules & Guardrails (AI Studio Optimized)

- **Build Verification:** Run `compile_applet` after `.kt`/`.kts`/`.xml` changes; report actual output before declaring complete.
- **Edit Batching:** Batch planned edits first; run `compile_applet` at end of change sequence.
- **Smart File Ops:** `view_file` before every edit. Write complete files first pass — no empty placeholders. Prefer `list_dir` over shell `ls`.
- **Shell Commands:** Never run `cd` (always pass `Cwd`). No `git push` unless explicitly kept local (`git add`, `git commit`).
- **Visual Assets:** `generate_image` for banners/illustrations/icons — `lowercase_snake_case` naming.
- **Web Search:** `search_web` to verify library syntax/API changes before acting.

## 🛑 Verify Before You Build (Three-Layer Approach)

To ensure accurate, complete, and resilient changes, **always** execute this three-layer validation sequence before declaring a task finished:

1. **Layer 1: Update MD (Documentation Sync)**
   - Before executing builds, synchronize the 6 core DOX files (`README.md`, `PLAN.md`, `ROADMAP.md`, `LOG.md`, `AGENTS.md`, `CONTEXT.md`) with the intended changes.
   - Update `PLAN.md` with current status and `LOG.md` with action records.
2. **Layer 2: Enable Tools (Automated Verification)**
   - **Build:** Run `compile_applet` to ensure compilation success.
   - **Test:** Run `./gradlew testDebugUnitTest` to ensure unit & Robolectric tests pass.
   - **Lint:** Run `lint_applet` or `./gradlew lint` if static analysis is required.
   - **Gate:** Use the `no-mistakes` pre-push pipeline to ensure codebase constraints are met.
3. **Layer 3: Human Validation Zones (Mandatory Stops)**
   - Stop and explicitly ask the user for confirmation when touching these zones:
     - Introducing breaking API or architecture changes.
     - Modifying `VaultSecurity` or Keystore implementations.
     - Adding new 3rd-party dependencies.
     - Deleting >20 lines of code.
     - Making significant shifts to data schemas (Room DB migrations).

## 🔐 Security Invariants (Crucial)

- Never log plain text secrets or API keys.
- Store sensitive values ONLY in `EncryptedSharedPreferences` (Android KeyStore backed).
- Keep secret input fields masked by default (`PasswordVisualTransformation`).
- Mark clipboard copies with `ClipDescription.EXTRA_IS_SENSITIVE` on API 33+.

## 💻 Coding Invariants (Ponytail)

- **Ponytail Hierarchy:** 1. Needs to exist? → 2. Already in codebase? → 3. Kotlin stdlib? → 4. Native Android feature? → 5. Existing dependency? → 6. One-liner? → Only then write new code.
- **Change Scope:** Edit only necessary lines — never touch unrelated code.
- **Safeguards:** Never skip validation, error handling, Keystore security, or accessibility.
- **Confirmation Required:** Ask user before introducing breaking changes, new dependencies, schema/API shifts, or deleting >20 lines.
- **Ambiguity Rule:** Ask for clarification — never guess.
- **Explanations:** 1-line summary max — no long essays.

## 📝 Documentation Rules (Check all 6 after meaningful edits)

- README.md  → Setup / usage changes
- PLAN.md    → Current task status
- ROADMAP.md → Milestones / feature status
- LOG.md     → Always append 1 dated entry line
- AGENTS.md  → Agent contracts / architecture changes
- CONTEXT.md → Key decisions & context

**Rule:** Match target file format/tone. If only LOG.md updated, append: "no other doc updates needed".

## Skills

use skill "find-skills" to look for skill on skill.sh if you needed skill to solve problem, blocked, improve, debugging, guidelines, etc - tell user before install.

## Issue workflow

- New issues start with `needs-triage`.
- After a finding is reproduced or validated and its scope is clear, add `ready-for-agent`.
- Security findings must also use the `security` label and be prioritized immediately.

## Agent Guidelines (Persona & Behavior)

- Adopt modern development practices (MVVM, Clean Architecture) within the minimal code constraints.
- Prioritize native platform libraries and existing dependencies over third-party libraries.
- Write clean, production-ready, self-documenting code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ak-a-ra/KeyNest](https://github.com/ak-a-ra/KeyNest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
