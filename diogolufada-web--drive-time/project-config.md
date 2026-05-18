---
trigger: always_on
description: FlutterFlow AI is a local workspace for creating and editing FlutterFlow apps with a coding agent.
---

# FlutterFlow AI Workspace

FlutterFlow AI is a local workspace for creating and editing FlutterFlow apps with a coding agent.

## Files

- `dsl/create.dart`
- `dsl/edit.dart`
- `test/app_test.dart`
- `references/`
- `patterns/`
- `PROJECT_CONTEXT.md`
- `context/`
- `generated_code/` — read-only snapshot of the Flutter code FlutterFlow generates from the project. Manifest at `generated_code/.flutterflow/export_manifest.json` maps each entity (page, component, action block, etc.) to its `primary_files`. Use this when debugging visual or runtime bugs the DSL alone cannot explain (overflow, layout, render errors, build failures).
- `.flutterflow/` (SDK-managed: run history, traces, workspace state, plus router config)

## Workflow

### Selector-first edit workflow

If the user pasted a `FlutterFlow AI Selector v1` block, use it before any broad page/component inspection:

1. Parse the pasted block for `project_id`, `scope_kind`, `scope_name`, `selector_path`, `node_key`, `node_name`, and `node_type`.
2. Run `flutterflow ai inspect <project_id> --page|--component <scope_name> --selector-path <selector_path> --dsl-json` to resolve the target widget.
3. Verify the returned `node_type` and `node_name` match expectations from the pasted block.
4. **If the user is reporting a visual or runtime bug** (overflow, layout, render error, exception, "looks wrong" / "doesn't fit"): before authoring the patch, read the generated Dart for the selector's scope.
   - Look up the entity in `generated_code/.flutterflow/export_manifest.json` by `name == scope_name` (or `key == node_key`).
   - Read its `primary_files` to see the actual widget tree, constraints, and styling Flutter is rendering.
   - The DSL is intent; the generated code is what is actually running. Overflow, an unbounded `Column` inside a `Row`, fixed sizes vs. `Expanded`, etc. are only visible there.
   - If `generated_code/` is missing or stale (`flutterflow ai codegen status` reports `stale`/`missing`), run `flutterflow ai codegen refresh` first.
5. Author the patch in `dsl/edit.dart` using `findByPath(...)`:
   ```dart
   app.editPage('PageName', (page) {
     page.findByPath('PageName.body[0].children[1]').update((patch) {
       // ...
     });
   });
   ```
6. Run `dart test`, then `flutterflow ai validate`, then `flutterflow ai run`.
7. If `--selector-path` fails, fall back to `--selector-key` with the `node_key` from the block.
8. Only do a broad `flutterflow ai inspect --page/--component` pass when the selector is stale or missing.

### General workflow

1. Start from the closest working examples in `references/`. Do not read the full API surface first unless the references are insufficient or you are blocked.
2. For edit work, inspect first with `flutterflow ai inspect <project-id>`.
3. Edit `dsl/create.dart` or `dsl/edit.dart`.
4. Update `test/app_test.dart` to match your changes (page names, component names, expected structure). The starter test references `StarterPage` — change it to match whatever you built.
5. Run `dart test`.
6. Run `flutterflow ai validate ...`.
7. **Execute the push** — this is NOT optional, always run this as the final step. Always include `--commit-message` with a short description of what changed:
   - **Create:** `flutterflow ai run dsl/create.dart --project-name "<name>" --commit-message "<what the app does>"`
   - **Edit:** `flutterflow ai run dsl/edit.dart --project-id "<id>" --commit-message "<what changed>"`
   - Use `--find-or-create` only as a retry/recovery option when a previous create run may already have created the remote project but the local workspace is not bound yet.
   - If the workspace is already bound to a project in `.flutterflow/workspace.json`, FlutterFlow AI will refuse plain create mode by default. Use `--allow-new-project` only when you intentionally want a second project from the same workspace.
8. After the **first successful push**, run `flutterflow ai refresh-context <project-id>` using the project ID from `.flutterflow/workspace.json` to generate `PROJECT_CONTEXT.md`.

## Design & Quality Rules

**These are mandatory for every create and edit script.**

### Theme first
Always set up a theme before building UI. Use `app.themeColor()`, `app.primaryFont()`, and `app.darkMode()` to define a coherent color palette. Never hardcode hex colors in widgets — use `Colors.primary`, `Colors.secondary`, `Colors.secondaryText`, etc. See `references/styled_profile_dsl.dart` for the pattern.

### Components for reuse
Extract repeated widget subtrees into `app.component()`. If a card, list item, or section appears more than once (or would appear in multiple pages), make it a component with typed `params:`. Prefer small, focused components over monolithic pages.

### Default values on params
Any `params:` entry whose `DslType` has no `.withDefault(...)` is treated as **required**. That has two consequences you must handle:

- **Component instances** must pass every required param at every call site (the SDK throws "Missing required parameter(s)" at compile time if you forget) — but those values are still arbitrary at runtime, so a `null`-bound state passed in will crash the page.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diogolufada-web/drive-time](https://github.com/diogolufada-web/drive-time) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
