---
trigger: always_on
description: These instructions are for AI coding agents working in this repository.
---

# AGENTS.md (Setup Your Mac, version 1.16.0)

These instructions are for AI coding agents working in this repository.

## Scope and priority

- Primary focus: `Setup-Your-Mac-via-Dialog.bash`
- Secondary files (edit only when required by the task): `Prompt-to-Setup-Your-Mac.bash`, `Validations/*`, docs
- Preserve behavior unless the user explicitly asks to change it

## Project context

`Setup-Your-Mac-via-Dialog.bash` is a production enrollment workflow script that:

- runs as `root`
- drives end-user UI with swiftDialog
- executes Jamf Pro custom policy events
- depends on guardrails (pre-flight checks, dependency checks, exit handling, logging)

Breakages can affect zero-touch onboarding and first-run usability. Optimize for operational safety.

## Non-negotiables

### Preserve SYM-Helper compatibility

Anything marked `[SYM-Helper]` in `Setup-Your-Mac-via-Dialog.bash` is externally managed by SYM-Helper.

- Do not remove `[SYM-Helper]` markers
- Do not rename, repurpose, or silently change semantics of `[SYM-Helper]` variables
- Do not change expected value formats for `[SYM-Helper]` variables without explicit request

### Maintain operational safety

- Keep dependency and environment checks intact (`root`, shell/runtime checks, swiftDialog/Jamf assumptions)
- Do not bypass failure paths or exit-code handling
- Do not break dialog launch/update/quit flow
- Do not reduce logging coverage in existing critical paths

## Editing rules for `Setup-Your-Mac-via-Dialog.bash`

- Keep script parameter numbering stable (`$4`, `$5`, etc.) unless explicitly requested
- Keep default parameter values backward compatible unless explicitly requested
- Reuse existing helpers/functions (`logMessage`, `runAsUser`, etc.) instead of ad-hoc patterns
- Prefer small, localized diffs over broad refactors
- Match existing Bash style and quoting conventions in surrounding code
- Use `lowerCamelCase` for new variable and function names unless matching an existing external contract (Jamf parameter labels, JSON keys, command flags, or `[SYM-Helper]` variables)
- Avoid introducing new external dependencies

## Behavioral invariants (must hold unless explicitly changed)

- Preserve dialog lifecycle: welcome dialog -> setup dialog updates -> finalize/failure messaging -> quit/completion action
- Preserve overall trigger flow for each step: policy execution/confirmation -> validation -> list item status update -> progress increment
- Preserve status semantics used in UI/logging (`pending`, `wait`, `success`, `fail`, `error`) and current user-facing statustext meanings
- Preserve failure accumulation behavior (`jamfProPolicyTriggerFailure`, `jamfProPolicyNameFailures`, `exitCode`)
- Preserve completion-action behavior for `Wait`, `Sleep`, `Log Out`, `Restart`, `Shut Down`, `Quit`, and debug-mode override behavior
- Preserve artifact cleanup behavior in completion and quit paths (command/json temp files, overlay assets, dialog log cleanup)

## Refactor boundaries

Refactor-friendly areas (default starting points):

- Pure data shaping and helper logic that does not change side effects
- Repeated string/JSON assembly where output can be kept identical
- Small helper extraction inside validation branches while preserving branch behavior

High-risk areas (edit only with explicit intent and targeted verification):

- `dialogUpdate*` call ordering and timing-sensitive UI updates
- `confirmPolicyExecution`, `validatePolicyResult`, and the main step/trigger loops
- `quitScript`, `completionAction`, launch daemon toggling, and cleanup sequencing
- Pre-flight checks that gate execution (`root`, shell/runtime, dependencies, logging)

## Runtime dependency contracts

- `jamf` unavailable: script must fail predictably through existing guardrails; do not add silent fallbacks
- swiftDialog unavailable or below minimum: preserve existing install/check behavior and messaging
- `runAsUser`/`launchctl` assumptions: keep command execution model intact for UI-facing actions
- Network-dependent operations (download estimates, webhooks, remote validations): preserve current failure-tolerant behavior and logging
- macOS-native tools (`PlistBuddy`, `scutil`, `fdesetup`, `system_profiler`) are part of current contract; do not replace with new deps

## Refactor strategy

Use incremental phases and verify after each phase:

1. Isolate pure helpers (string transforms, value parsing, repeated message assembly)
2. Refactor function internals without changing inputs/outputs or call order
3. Refactor orchestration only after helper-level behavior is stable
4. Stop and report when a planned change requires behavioral deviation

## Validation and policy changes

When modifying policy/validation behavior:

- Preserve existing `policyJSON` shape unless task requires a schema change
- Keep local/remote validation behavior consistent with current conventions
- Avoid changing user-visible policy names/labels unless requested
- Document behavior changes in `CHANGELOG.md`

## Logging and error handling

- Use the script's logging conventions; avoid unstructured `echo` for production paths
- Ensure errors still feed expected failure handling and final status reporting
- Do not swallow non-zero statuses without a clear reason and log message


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [setup-your-mac/Setup-Your-Mac](https://github.com/setup-your-mac/Setup-Your-Mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
