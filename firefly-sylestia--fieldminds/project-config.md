---
trigger: always_on
description: This file is part of the **DOX framework** defined in `master.md`. All agents MUST follow the DOX hierarchy:
---

# FieldMind Project — Root AGENTS.md (DOX Rail)

## DOX Framework

This file is part of the **DOX framework** defined in `master.md`. All agents MUST follow the DOX hierarchy:

1. **`master.md`** — DOX framework definition (core contract, read/edit workflow, style, closeout)
2. **`AGENTS.md`** (this file) — Project-wide DOX rail: environment rules, workflow, Prompt.md, What's New guidance
3. **Child AGENTS.md files** — Domain-specific contracts for each subtree

**Every agent MUST read `master.md` + the root `AGENTS.md` + the nearest child AGENTS.md along every path they touch before editing.** Do not rely on memory.

## Purpose

Top-level instruction file for all AI agents (Codebuff/Buffy and spawned sub-agents) working on the FieldMind Android project. Project-wide rules, global preferences, and the top-level Child DOX Index.

## Critical Environment Rules

### ❌ NEVER RUN COMPILE OR BUILD COMMANDS

**Do not run any Gradle compile, build, assemble, or lint commands in this environment.** This includes but is not limited to:

- `./gradlew assemble*`
- `./gradlew compile*`
- `./gradlew build`
- `./gradlew lint`
- `./gradlew ksp*`
- `./gradlew ktlint*`
- `./gradlew test`
- `./gradlew check`

**Reason:** The development environment (IDX/workspace) does not have the full Android SDK, NDK, or build tools configured. Running these commands will fail. All compilation and build validation is handled by CI (GitHub Actions) on push.

### 🛡️ COMPILE-SAFETY RULES (read before ANY edit)

These rules were derived from a postmortem of actual CI compilation failures
(see `docs/CI_ERROR_POSTMORTEM.md`). Every error was avoidable. Follow these
rules to prevent repeating them.

1. **READ BEFORE WRITING** — Before constructing any entity, ViewModel,
   settings, or data class constructor call, **read the actual data class
definition file**. Do not assume parameter names from memory.

2. **CHECK COMPOSE BOM** — Before using a Material3 API, check
   `gradle/libs.versions.toml` for the Compose BOM version. Cross-reference
   with the Material3 changelog to confirm the API exists in that version.
   (E.g. `Card(onClick=…)` requires Material3 1.2+, `tonalElevation`
   requires a later version.)

3. **NON-COMPOSABLE LAMBDAS** — `BackHandler`, `onClick`, `onValueChange`,
   `onCheckedChange`, `LaunchedEffect` key lambdas, and any
   `callback: () -> Unit` are **NOT** @Composable contexts. Do not call
   `remember`, `mutableStateOf`, `LocalFoo.current`, or any @Composable
   function inside them. Extract those calls to the enclosing @Composable
   scope.

4. **NO SED FOR KOTLIN** — Never use `sed -i` to insert multiline Kotlin code.
   Always use `str_replace` with exact old/new string matching. If you must
   use a terminal command for insertion, verify the output afterward.

5. **IMPORTS** — When removing an import, verify **all references** to the
   type are also removed/updated. `CardColors`, `CardElevation`,
   `RoundedCornerShape`, `Shape`, and similar Material3 types are often used
   in function signatures — removing their imports while they're still
   referenced causes compile failures.

6. **MODIFIER ORDER** — When stacking interaction modifiers, order matters:
   press-detection modifiers (`expressiveCardPress`, `pointerInput`) come
   **before** click-consumption modifiers (`clickable`, `combinedClickable`).
   The first modifier in the chain has priority for pointer events.

7. **CANVAS PARAMETER NAMES** — Never name a parameter `size` in a function
   that contains a `Canvas {}` block. Use `iconSize`, `imageSize`,
   `tileSize`, etc. to avoid shadowing `DrawScope.size`.

8. **COMPOSABLE IS A FUNCTION** — `@Composable` only applies to functions,
   never to property getters. Use `@Composable fun foo(): Type` not
   `val foo: Type @Composable get()`.

9. **VERIFY ONE-CYCLE** — After pushing a CI fix, wait for the CI result.
   If the CI log shows NEW errors in files you didn't touch, the previous
   fix may have been incomplete. Do not assume a commit is final until CI
   passes.

10. **TEST SMOKE** — For entity/data-layer changes, the
    `DevFullAppTestRunner` in Developer Settings can verify constructors,
    settings toggles, and database operations without a full Gradle build.

### ✅ DO COMMIT AND PUSH CHANGES PROPERLY

Always commit and push changes using proper git workflow:

1. **Stage changes:** `git add -A` (or specific files)
2. **Commit with descriptive message:** `git commit -m "type: concise description of changes"`
3. **Push:** `git push`

Follow conventional commit format: `feat:`, `fix:`, `refactor:`, `docs:`, `style:`, `chore:`, etc.

## Prompt.md — Research & Analysis Tracking

`Prompt.md` at project root is the running log of the current request. See `Prompt.md` itself for its own rules. Agents must update Prompt.md when:
- Starting a new request (replace entirely with fresh analysis)
- A request is interrupted or half-done (capture progress, remaining work, decisions)
- A request is completed (add completion summary)

## General Workflow

1. **Read DOX chain** — `master.md` → `AGENTS.md` → child AGENTS.md along every path you touch
2. **Read Prompt.md** — check for existing context or half-finished work

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firefly-sylestia/FieldMinds](https://github.com/firefly-sylestia/FieldMinds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
