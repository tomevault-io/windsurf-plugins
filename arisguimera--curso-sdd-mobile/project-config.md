---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository.
All paths in this document are relative to the repository root.

## Required reading

Before planning, reviewing, or modifying this project, read
[docs/GENERIC_RULES.md](docs/GENERIC_RULES.md) in full and apply it alongside
this file. The link is an explicit reading requirement; do not assume your
tool automatically imports linked Markdown files.

GENERIC_RULES.md contains reusable working rules. This file adds the project's
Android conventions and spec-driven workflow. For repository conventions,
project-specific rules refine the generic defaults. Neither file overrides
the user's explicit instructions or the agent's higher-priority instructions.
If documents conflict in a way that affects behavior or scope, clarify the
conflict before implementing the affected part.

## Project

Native Android app built with Kotlin and Jetpack Compose as a course project
("Curso SDD Mobile", spec-driven development).

- Single module: `app`.
- Package: `com.aristidevs.cursopremiumandroid`.
- Baseline functionality: fetch a dog catalog and dog details from a static
  JSON API and render them with Compose.
- The existing Dog feature is the reference for new features. Inspect its
  implementation before extending the app.

## Commands

Run commands from the repository root using the Gradle wrapper.

```bash
./gradlew :app:assembleDebug          # Build the debug APK
./gradlew :app:test                  # Run local JVM tests across variants
./gradlew :app:testDebugUnitTest      # Run local JVM tests for debug
./gradlew :app:connectedDebugAndroidTest  # Requires a device/emulator
./gradlew :app:lintDebug              # Run Android lint for debug

# Run one local test class; replace SomeTest with an existing class
./gradlew :app:testDebugUnitTest --tests "com.aristidevs.cursopremiumandroid.SomeTest"
```

Local tests live in `app/src/test`; instrumented tests live in
`app/src/androidTest`. If build variants change, inspect the available Gradle
tasks and use the appropriate variant-specific task.

There is no separate ktlint/detekt configuration in the baseline repository.
Use the existing configuration; do not introduce a new quality tool as an
unrelated change.

For code changes, run `:app:assembleDebug`, `:app:testDebugUnitTest`, and
`:app:lintDebug` before reporting completion. Run relevant instrumented tests
and manual checks when acceptance criteria require device behavior. For
documentation-only changes, verify content and references without requiring
an Android build. Report any checks that could not run and the reason.

## Spec-driven workflow

### Reference documents

Each template carries its own instructions for the agent, its section
structure, and its identifier scheme. Read the template in full and follow it;
this file does not restate its content and must not contradict it.

- `docs/SPEC_TEMPLATE.md`: copy to the feature's `SPEC.md` when creating or
  updating a specification. It owns the spec's sections, its `RF-` requirement
  and `CA-` acceptance-criteria identifiers, and its approval states.
- `docs/PLAN_TEMPLATE.md`: copy to the feature's `PLAN.md` once the
  specification is approved. It owns the plan's sections and approval states.
  Reference requirements and criteria by their spec identifiers.
- `docs/MOBILE_GUIDELINES.md`: consult when writing or reviewing requirements,
  acceptance criteria, the technical plan, and mobile validation. Consider
  lifecycle, state retention, connectivity, persistence, UI states, navigation
  and interruptions, forms, screen adaptation, accessibility, permissions,
  performance, background work, privacy, and internationalization. Apply only
  relevant items; clarify undefined product behavior instead of inventing it.

Preserve each template's structure and its embedded comments in the copy. Do not
rewrite the shared templates for an individual feature.

### Feature documents

Keep each feature's documents together:

- `docs/features/<feature-name>/SPEC.md`
- `docs/features/<feature-name>/PLAN.md`
- `docs/features/<feature-name>/TASKS.md`

Use an existing feature directory when continuing its work.

### Sequence

Each stage is gated by the previous document's state. A document is only
`Aprobada`/`Aprobado` when the user says so: a complete document is not an
approved one, and the agent never changes that state on its own.

1. **Specification:** complete `SPEC.md` from `docs/SPEC_TEMPLATE.md`,
   collaboratively and section by section, following the template's own
   instructions. Do not start the plan until the user approves the spec.
2. **Plan:** write `PLAN.md` from `docs/PLAN_TEMPLATE.md` for the approved
   specification, following the template's own instructions. Additionally,
   record whether subagents are needed and their bounded responsibilities; do
   not assume delegation is required or available. Do not start tasks until
   the user approves the plan.
3. **Tasks:** derive `TASKS.md` from the approved plan. There is no shared
   template for it, so this file defines it: small, ordered, verifiable
   checkboxes, each with an identifier, objective, scope, dependencies, the
   spec criteria it resolves, and its validation method. Keep tasks concise

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArisGuimera/Curso-SDD-Mobile](https://github.com/ArisGuimera/Curso-SDD-Mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
