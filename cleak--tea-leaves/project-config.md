---
trigger: always_on
description: Operational instructions for coding agents in `C:\Projects\Godot\tea-leaves`.
---

# AGENTS.md

Operational instructions for coding agents in `C:\Projects\Godot\tea-leaves`.

## Mission

Ship correct Godot features quickly with automatic verification. Do not wait for the user to ask for tests, lint, UID fixes, or runtime screenshots.

## Project Profile

- Godot 4.6+ Mono.
- Gameplay code: C# only.
- GDScript: editor tooling and tiny glue scripts only.
- Physics: Jolt.
- Renderer: Forward Plus.
- Target platform: Windows (D3D12).

## Hard Rules (Non-Negotiable)

1. Always run applicable tools automatically after edits.
2. Always create or update tests for behavior changes, bug fixes, and non-trivial refactors.
3. Always run those tests, plus broader suites required by changed file types.
4. On any non-documentation change, run the global verification gate before finalizing.
5. Always validate and repair UID/GUID issues after scene/resource/script/shader edits.
6. Always attempt runtime screenshot verification for gameplay-facing changes.
7. Never silently skip a required check. If blocked, state the blocker explicitly.

## Mandatory Autopilot Workflow

Follow this sequence on every coding task unless it is explicitly documentation-only.

1. Classify changed files and affected behavior.
2. Add/update tests first for behavior changes (red/green when fixing bugs).
3. Run fast targeted checks while iterating.
4. Run the global verification gate (non-doc changes) plus all required checks from the matrix below before finalizing.
5. For gameplay-impacting changes, run DevTools runtime validation + screenshot capture.
6. Report commands run and pass/fail outcomes in the final message.

## Test Creation Policy (Explicit)

### Required

- New feature: add tests that cover expected behavior.
- Bug fix: add a failing test first (or tighten an existing test), then fix, then rerun.
- Refactor with behavior risk: add characterization/regression coverage.
- Scene/interaction changes: add or update automated coverage, then verify at runtime with input simulation and screenshot.

### Allowed Exceptions

If a test cannot be added, agent must explain why in final output using one of:
- `external_tooling_blocked`
- `runtime_dependency_missing`
- `legacy_test_harness_gap`

If no exception applies, test creation is mandatory.

### Minimum Test Delta Rule

- For behavior changes and bug fixes, at least one test must be added or strengthened.
- "Strengthened" means tighter assertions, broader coverage, or a new edge case in an existing test.
- If neither is done, the final report must include a blocker code from Allowed Exceptions.

## Global Verification Gate (Mandatory for Non-Docs Changes)

Run all commands below for any code/scene/resource/tooling change:

```powershell
dotnet build -warnaserror
dotnet test
pwsh ./tools/test.ps1
pwsh ./tools/godot.ps1 --headless --script res://tools/lint_project.gd
```

Do not mark the task complete until this gate passes, or blockers are explicitly documented.

## Required Automatic Checks by Change Type

Run every row that applies.
Rows are additive to the Global Verification Gate; they do not replace it.

| Changed area | Required commands |
|---|---|
| `*.cs`, `*.csproj`, `*.sln` | `dotnet restore` (when needed), then Global Verification Gate |
| `*.tscn`, `*.tres`, `*.res`, `*.uid`, `project.godot` | Global Verification Gate; if only specific scenes changed, also run targeted scene lint with `-- --scene res://...` during iteration |
| `*.gdshader` | Global Verification Gate + `pwsh ./tools/godot.ps1 --headless --script res://tools/lint_shaders.gd` |
| `*.gd` | `gdlint <each_changed_file.gd>` + semantic parse check `pwsh ./tools/godot.ps1 --headless --check-only --script res://path/to/file.gd`; if test files changed, `pwsh ./tools/lint_tests.ps1`; then Global Verification Gate |
| Input/tooling changes (`tools/setup_input_actions_cli.gd`, input maps) | re-run setup script, then Global Verification Gate |
| Gameplay behavior (movement, camera, combat, interaction, UI flow) | Global Verification Gate + DevTools runtime loop: `ping`, relevant `input` simulation, `screenshot`, `validate-all`, `performance`, `input clear` |

## UID/GUID Integrity Policy (Mandatory)

Godot UID integrity is part of correctness.

1. After hand-editing scenes/resources or adding scripts/shaders, run:
   - `pwsh ./tools/godot.ps1 --headless --script res://tools/lint_project.gd`
2. If lint rewrites UIDs, update any stale `uid://...` references immediately.
3. Re-run lint until clean.
4. Always include generated `*.uid` files in the change set.

## Runtime Verification and Screenshots (Mandatory for Gameplay Changes)

For any gameplay-visible change, do all of the following automatically:

1. Ensure game is running: `pwsh ./tools/godot.ps1` (if not already running).
2. Verify DevTools: `python tools/devtools.py ping`.
3. Simulate relevant actions using `python tools/devtools.py input ...` or a sequence file.
4. Capture at least one screenshot:
   - `python tools/devtools.py screenshot --filename "<feature>_<state>.png"`
5. Run runtime validation:
   - `python tools/devtools.py validate-all`
   - `python tools/devtools.py performance`
6. Clear inputs:
   - `python tools/devtools.py input clear`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cleak/tea-leaves](https://github.com/cleak/tea-leaves) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
