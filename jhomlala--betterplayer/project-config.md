---
trigger: always_on
description: - **Commit/Push Policy**: NEVER commit or push changes automatically. ALWAYS wait for explicit user approval before performing any Git commit or push operations.
---

# Better Player AI Rules

## Development Workflow
- **Commit/Push Policy**: NEVER commit or push changes automatically. ALWAYS wait for explicit user approval before performing any Git commit or push operations.
- **Git Hygiene**:
  - NEVER create or change branches automatically. ALWAYS wait for explicit user instruction on which branch to use.
  - ALWAYS `git fetch origin master` and `git rebase origin/master` before starting any new task to ensure you are working on the latest code.
  - Ensure your branch history is clean and only contains commits relevant to the current task.
  - Use standard branch naming conventions based on the task type:
    - `feat/` for new features.
    - `fix/` or `bugfix/` for bug fixes.
    - `docs/` for documentation updates.
    - `refactor/` for code refactoring.
    - `test/` for adding or updating tests.
    - `chore/` for maintenance tasks or dependency updates.
- **Post-Implementation Steps**: If you have changed any Dart code, ALWAYS run the following commands in the workspace root after completion of a plan or task:
  - `dart format .`
  - `flutter analyze .`
  - Ensure all packages in `packages/` are consistent.
- **Error Resolution**: If `flutter analyze` reports issues, they MUST be fixed immediately before concluding the task.
- **PR Completeness**: Before considering a task or bug fix finished, ensure the work is ready to be merged. This includes:
  - Adding a relevant entry to `CHANGELOG.md` under the `## Unreleased` header. If the header does not exist, create it at the very top of the file (following the Changelog Guidelines).
  - Ensuring all tests pass and `flutter analyze` is clean.
  - Providing clear verification steps and, for bugs, the reproduction data source.

## Code Style & Linting
- **Standard**: Follow `package:very_good_analysis`.
- **Formatting**: Adhere to standard Dart formatting.
- **Specific Rules**:
  - Prefer single quotes over double quotes.
  - Require trailing commas for multi-line arguments and collections.
  - Maintain the existing suppressions in `analysis_options.yaml` for specific project needs (e.g., `public_member_api_docs: false`).
  - **Named Parameters**: Use named parameters for all functions, methods, and constructors if:
    - They have 2 or more parameters.
    - They have only 1 parameter and that parameter is a `bool`.
  - **Widget Creation**: NEVER create widgets using helper methods (e.g., `Widget _buildSomething()`). ALWAYS create them as separate `StatelessWidget` or `StatefulWidget` classes, or define the widget tree directly within the `build` method. This ensures better performance, cleaner code, and correct lifecycle management.
  - **Logging**: ALWAYS use `BetterPlayerUtils.log` for logging instead of `print` or `debugPrint`. This ensures that logs are correctly handled by the project's logging mechanism.

## Testing
- **Async Operations**: Always `await` asynchronous calls in tests (e.g., `setupDataSource`, `play`, `pause`, `seekTo`).
- **Mocking**: Use `BetterPlayerMockController` and `MockVideoPlayerController` for unit tests.
- **Verification**: ALWAYS run tests using the following command at the workspace root or within the relevant package directory to efficiently identify failures and avoid token limit issues:
  - Use `flutter test` within a specific package directory (e.g., `cd packages/better_player; flutter test`).
  - For workspace-wide testing, use:
  ```powershell
  $names=@{}; Get-ChildItem -Path packages -Directory | ForEach-Object { cd $_.FullName; if (Test-Path test) { flutter test --machine | ForEach-Object { if ($_ -match '^{.*}$') { $_ | ConvertFrom-Json } } | ForEach-Object { if($_.type -eq "testStart"){$names[$_.test.id]=$_.test.name} elseif($_.type -eq "error"){[PSCustomObject]@{test=$names[$_.testID]; error=$_.error; package=$_.FullName}} } } }; cd ../.. | ConvertTo-Json -Compress
  ```

## Project Structure
- **Test Organization**: The `test/` directory MUST mirror the `lib/src/` directory structure.
  - Example: `lib/src/core/` -> `test/core/`
  - Helpers and mocks should be placed in `test/helpers/`.
  - Test files should be named `<original_file_name>_test.dart` or reflect the component they test.
- **Artifacts**: Never track the `.artifacts/` directory in Git. It is already added to `.gitignore`.
- **Example App**: When changing core library code, check if the `example` app needs updates or if its tests/analysis are affected.

## E2E Testing (Maestro)
- **Rules Reference**: ALWAYS follow the guidelines in [rules/maestro.md](file:///C:/Users/jhoml/betterplayer/rules/maestro.md) when writing or updating E2E tests.
- **Stability First**: Prioritize using `Semantics(identifier: ...)` in Flutter and `id` selectors in Maestro. Avoid text-based selectors or hardcoded coordinates.
- **iOS Focus**: Ensure all E2E flows are verified on the iOS Simulator, as this is the primary focus for E2E reliability.

## Architecture
- **Plugin-First Principle**: When working on new features or refactoring existing code, prioritize a **plugin-based architecture**. The goal is to keep the core library lean and extend functionality via plugins rather than purely working within the core.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhomlala/betterplayer](https://github.com/jhomlala/betterplayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
