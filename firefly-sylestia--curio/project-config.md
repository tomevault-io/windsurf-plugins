---
trigger: always_on
description: This file is part of the **DOX framework** defined in `master.md`. All agents MUST follow the DOX hierarchy:
---

# Curio Project — Root AGENTS.md (DOX Rail)

## DOX Framework

This file is part of the **DOX framework** defined in `master.md`. All agents MUST follow the DOX hierarchy:

1. **`master.md`** — DOX framework definition (core contract, read/edit workflow, style, closeout)
2. **`AGENTS.md`** (this file) — Project-wide DOX rail: environment rules, workflow, Prompt.md, What's New guidance
3. **Child AGENTS.md files** — Domain-specific contracts for each subtree

**Every agent MUST read `master.md` + the root `AGENTS.md` + the nearest child AGENTS.md along every path they touch before editing.** Do not rely on memory.

## Purpose

Top-level instruction file for all AI agents (Codebuff/Buffy and spawned sub-agents) working on the Curio Android project. Project-wide rules, global preferences, and the top-level Child DOX Index.

**⚠️ SCOPE: This project's active workstream is the Android app (`app/`) plus
the account site (`auth-web/`, which is live infrastructure for that app's
accounts, not a port). `web/` and `desktop/` are separate projects on hold — do
not touch them unless the user explicitly asks (see the 🔒 Scope section
below).**

## ❓ ASK WHEN UNSURE

If you understand the user's request less than ~80%, **ask for confirmation
before doing anything**. Do not guess, do not assume, do not pick the most
plausible interpretation and run with it. A wrong guess wastes a full cycle
(edit → review → commit → push → CI → revert) and can ship an unwanted
change.

**Durable user preference — always ask before DELETING or REPLACING
anything:** Before removing an existing feature, behavior, UI element, or
code path — and before deleting, replacing, or overwriting ANY file, data
entry, or content (topic JSON entries, strings, assets, docs) — ask the
user for confirmation first. Refinements may change implementation
details only when the existing user-visible behavior is preserved; if
removal or replacement is part of the proposed fix, pause and ask. When
in doubt, use the ask_user tool to clarify the request, and only proceed
once the user confirms.

This rule covers ambiguous phrasing, missing context, conflicting
instructions, and any request where multiple readings would lead to
different implementations. Spawned sub-agents don't have the ask_user tool
— when they hit this uncertainty they must report it back to the parent
agent, who asks the user.

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

### 👀 NEVER WAIT ON A CI RUN — ASK WHETHER IT IS GOING, AND CARRY ON (user directive, 2026-09-21)

**Do NOT sit and watch a CI run.** While a run is `in_progress`, keep working:
answer the member, do the next item on the plan, start the next fix. Watching a
build burns the session and delivers nothing.

Check a run only to make a DECISION, and only with a single quick call
(`gh run list --limit 3`):

1. **The run is `in_progress`** → say so only if it is what was asked, and carry
   on with the work. Do not poll it again in the same task.
2. **The run has `failed`** → read its errors (`gh run view <id> --log-failed`),
   fix them, PUSH the fix, and then answer the member. The one moment a run must
   be looked at is just before a push, so a red build never becomes the pushed
   state twice in a row.
3. **The run is `success`** → nothing to do; keep working.

Answering the member NEVER waits on CI. If a fix was just pushed and the run is
still going, say what was pushed and what it addresses — the result is the next
session's business, or this session's if the member asks.

### 🛡️ COMPILE-SAFETY RULES (read before ANY edit)

These rules were derived from actual CI compilation failures. Every error was avoidable. Follow these rules to prevent repeating them.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firefly-sylestia/Curio](https://github.com/firefly-sylestia/Curio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
