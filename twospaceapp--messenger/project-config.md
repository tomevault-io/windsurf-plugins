---
trigger: always_on
description: This guide defines repository-wide instructions for coding agents working with the TwoSpace Flutter client.
---

# Agent Guide for TwoSpace Messenger

This guide defines repository-wide instructions for coding agents working with the TwoSpace Flutter client.

## Mission

- Treat this repository as a Flutter client first, not as a place to compensate for missing backend behavior.
- Prefer root-cause fixes over UI-only masking.
- Keep changes local, deliberate, and easy to review.
- Do not invent protocol capabilities that the Aegis server does not expose.

## Repository Shape

- `lib/core/` contains shared infrastructure such as config, routing, protocol client, services, and reusable widgets.
- `lib/features/` is feature-first and should contain most app-specific behavior.
- `lib/core/network/aegis/` and `lib/features/auth|chat/data/services/` are the most sensitive protocol paths.
- `lib/l10n/` contains localization inputs and generated localization outputs.
- `tool/` contains probes and developer utilities, not production app code.
- `test/` contains automated tests.
- `.github/workflows/` contains CI and should only be changed for build, validation, or release-flow tasks.

## Working Style

- Read the affected path before editing it.
- Do not behave as if you understood a code path before reading the relevant files.
- Prefer changing an existing service, notifier, DTO, or widget over adding a parallel abstraction.
- Keep package imports absolute: `package:two_space_app/...`.
- Preserve existing architecture unless the task explicitly requires a structural change.
- Do not overwrite unrelated user edits.

GOOD:

- Read the service, provider, and screen involved in a bug before proposing a fix.
- Say clearly when a behavior still depends on code or protocol you have not inspected yet.
- Ask in simple Russian before changing a distinctive UI solution that may be intentional.
- Extend `AegisChatService` if chat/profile behavior already lives there.
- Add a small helper in an existing notifier if the state already belongs to that notifier.
- Reuse an existing shared widget or theme helper before adding another one.

BAD:

- Guess how a feature works from one widget name and start editing immediately.
- Claim a root cause without reading the code path that actually produces the data or error.
- Replace a distinctive interaction pattern with a generic one without user approval.
- Create `NewProfileService` because the existing service looks inconvenient.
- Add local widget state that duplicates a Riverpod source of truth.
- Work around a protocol limitation by storing fake permanent data only on the client.

## Build And Validation

Run commands from the repository root.

Preferred validation order:

1. Validate changed files first: `flutter analyze <files>`
2. Run focused tests when relevant: `flutter test <target>`
3. Use broader validation only when the task actually needs it

Project commands:

- Install dependencies: `flutter pub get`
- Regenerate localizations: `flutter gen-l10n`
- Regenerate generated Dart code and env bindings: `dart run build_runner build -d`

If `.env` changes, regenerate Envied output with `dart run build_runner build -d`.

GOOD:

- Run `flutter analyze` only for touched files after a focused Dart/UI change.
- Run `flutter gen-l10n` after adding or changing ARB strings.
- Run `dart run build_runner build -d` after changing `freezed`, `json_serializable`, `riverpod`, or `envied` inputs.

BAD:

- Skip regeneration after changing annotated models and then patch generated output by hand.
- Run large validation by default when a targeted check is enough.
- Modify CI because a local command was inconvenient.

## Generated Files And Build Outputs

- Do not hand-edit generated files such as `*.g.dart`, `*.freezed.dart`, `lib/core/config/env.g.dart`, or platform-generated registrant/plugin files unless the task is explicitly about generation or tooling.
- Do not reintroduce CI checks that fail on platform-generated noise alone.
- Regenerate outputs from source inputs instead of patching generated artifacts.

GOOD:

- Edit the annotated source file and rerun generation.
- Treat generated plugin and CMake outputs as derived artifacts.

BAD:

- Patch a `*.g.dart` file because it is faster.
- Add a workflow step that fails whenever local generation order changes in platform-generated files.

## Architecture Rules

- Keep business logic out of presentation widgets when possible.
- Reuse current `AegisAuthService`, `AegisChatService`, protocol DTOs, and client code before adding wrappers.
- Follow Riverpod patterns already used by the repo.
- Keep navigation aligned with `GoRouter` and existing route constants.
- Prefer one clear source of truth for data ownership.

GOOD:

- Let the service own protocol interaction and cache synchronization.
- Keep widgets focused on rendering, user interaction, and thin state wiring.
- Update the current provider or notifier instead of adding a second cache.

BAD:

- Put reconnect logic directly inside a screen widget.
- Add a second in-memory profile map when the service already owns profile data.
- Trigger protocol requests from multiple layers for the same state change without coordination.

## Aegis Protocol Rules

- The server session is bound to the active TCP connection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TwoSpaceApp/messenger](https://github.com/TwoSpaceApp/messenger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
