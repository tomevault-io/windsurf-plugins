---
trigger: always_on
description: Messenger-style whisper UI addon for World of Warcraft: Retail, the Classic flavors (Vanilla, TBC, Wrath, Cata, Mists) and WoW: Forever. One TOC (`WhisperMessenger.toc`) carries every flavor's `## Interface-*` line.
---

# WhisperMessenger

Messenger-style whisper UI addon for World of Warcraft: Retail, the Classic flavors (Vanilla, TBC, Wrath, Cata, Mists) and WoW: Forever. One TOC (`WhisperMessenger.toc`) carries every flavor's `## Interface-*` line.

## Tech Stack

- **Lua 5.1** (WoW runtime) — no Lua 5.2+ features (no `goto`, no `table.unpack` without compat)
- **WoW API** — Blizzard's frame/widget system, C\_ namespaced APIs
- **Flavor differences** go through `Core/FlavorCompat.lua` — never assume a Retail-only API exists
- **StyLua** — formatter (`stylua.toml`)
- **Luacheck** — static analysis (`.luacheckrc`)

## Code Conventions

- **2-space indentation, spaces** (not tabs)
- **PascalCase** for modules/classes: `ConversationStore`, `TableUtils`
- **camelCase** for functions/variables: `buildContacts`, `refreshWindow`
- **snake_case** for theme constants: `bg_primary`, `text_secondary`
- **Module pattern**: every file starts with `local addonName, ns = ...` plus the `if type(ns) ~= "table" then ns = {} end` guard, and ends with `ns.ModuleName = ModuleName; return ModuleName`
- **Prefix unused args with `_`**: `_self`, `_event`, `_conversation`
- **Access WoW globals via `_G.`**: `_G.CreateFrame`, `_G.C_ChatInfo` — keeps the dependency on globals explicit and testable

## Linting

```bash
# Check (CI-safe) — always use the project script, not bare tool commands
bash scripts/lint.sh

# Auto-format + check
bash scripts/lint.sh --fix
```

Luacheck handles semantics (unused vars, undefined globals, shadowing). StyLua handles formatting (line length, spacing, alignment). Both must pass clean before committing — they are the only lint gates CI runs.

`lint.sh` also runs LuaLS diagnostics, which already report ~300 old problems, so the script exits 1 even on a clean tree. Don't add new LuaLS problems: compare the count against a stash of your changes.

Missing tools on Windows: `powershell -ExecutionPolicy Bypass -File scripts/setup-lint-tools.ps1` installs them into `.tools/`.

When adding new WoW API globals, add them to `.luacheckrc` under `read_globals`.

## Tests

```bash
# If lua is available:
lua tests/run.lua tests/path/to/test_file.lua

# If lua is not available (Windows), use the Python+lupa harness (pip install lupa):
python scripts/run_test.py tests/path/to/test_file.lua
```

Tests run with plain Lua — no WoW runtime needed. WoW APIs are stubbed via `tests/helpers/fake_ui.lua` and the `tests/helpers/fake_ui/` folder. Run all tests:

```bash
find tests -type f -name 'test_*.lua' | sort | while read -r f; do python scripts/run_test.py "$f"; done

# Release script tests (Python):
python -m unittest tests.scripts.test_gen_patch_notes tests.scripts.test_promote_changelog
```

The release pipeline minifies the shipped Lua and runs every test again on the minified code, so neither code nor tests may depend on comments or exact source formatting.

## Development Workflow — TDD (Red-Green-Refactor)

Every change follows test-driven development. No exceptions.

1. **Red** — Write a failing test first. The test must fail for the right reason (missing behavior, not a syntax error). Run it and confirm the failure.
2. **Green** — Write the minimum code to make the test pass. No more, no less. Run the test and confirm it passes.
3. **Refactor** — Clean up the implementation and the test. Remove duplication, improve naming, simplify logic. Run all tests to confirm nothing broke.

Rules:

- **Never write production code without a failing test that demands it.**
- **One behavior per test** — each test should verify a single expectation.
- **Test file mirrors source file** — `Model/ConversationStore.lua` → `tests/model/test_conversation_store.lua`
- **Test names describe behavior** — `test_mark_read_resets_unread_count`, not `test_mark_read_1`
- **Run the relevant test file after Red and Green steps.** Run all tests after Refactor.
- **Lint after Refactor** — run `bash scripts/lint.sh` before considering the cycle complete.

## File Size & Modularity

- **Target ~300 lines per file.** If a file grows past 300 lines, split it into focused sub-modules.
- **One responsibility per file** — a file that does two things should be two files.
- **Extract early** — when adding code would push a file over 300 lines, extract a new module before continuing.
- **Applies to both production and test files.**

## Adding a File

- **Add it to `WhisperMessenger.toc`, after everything it depends on.** TOC order is load order. WoW: Forever has a global `require` that throws "Invalid import", so a module's `ns.X or require(...)` fallback must never be reached in game. `tests/integration/test_toc_load_order.lua` guards this.
- **Folder modules** (`UI/Theme/`, `UI/Composer/`, …) have an `init.lua` that only exists so tests can `require` the folder. It is not in the TOC; add any new one to the `ignore:` list in `.pkgmeta`.
- **New top-level files that aren't addon code** (docs, configs) go in `.pkgmeta` `ignore:` so they don't ship in the zip.

## Localization

- English strings are the keys: `Localization.Text("Mark all as read")`. There is no enUS catalog.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [F0rty-Tw0/WhisperMessenger](https://github.com/F0rty-Tw0/WhisperMessenger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
