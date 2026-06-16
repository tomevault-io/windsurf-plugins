---
trigger: always_on
description: The full project reference lives in **`docs/directory.md`**. It covers architecture, directory map, backend/frontend module reference, data flow, flag system, chat template presets, Quick Launch, Chat, HF downloader, tunnel, auto-update, sampler presets, metrics, MCP, reasoning support, custom launch args, configuration search, smoke tests, and native file picker.
---

# AGENTS.md

## Project Reference

The full project reference lives in **`docs/directory.md`**. It covers architecture, directory map, backend/frontend module reference, data flow, flag system, chat template presets, Quick Launch, Chat, HF downloader, tunnel, auto-update, sampler presets, metrics, MCP, reasoning support, custom launch args, configuration search, smoke tests, and native file picker.

The test reference lives in **`docs/tests.md`**. It lists the frontend/backend test commands, test file purposes, and when to use focused unit tests versus browser smoke tests.

Read `docs/directory.md` before starting any task.

## UI State Sync Rule

When the same setting appears in more than one place in the UI, all instances must stay linked.

Examples:
- A setting shown in both `Configure` and `Quick Launch`
- Any duplicated model, template, sampler, or launch flag control across tabs

Required behavior:
- All duplicate controls must read from the same underlying state object.
- Changing the setting in one tab must immediately update the matching control in every other tab.
- Command preview / launch args must be generated only from the shared underlying state, never from per-tab copies.
- Avoid separate option lists for the same setting. Reuse the same flag definition or shared source list whenever possible.
- Prefer one shared setter function for each shared setting so updates, UI refresh, and launch-arg sync happen in one place.

Anti-patterns to avoid:
- Maintaining a custom dropdown list in one tab while another tab uses the real flag enum
- Having "helper" controls that do not call the same setter as the main control
- Letting one tab keep its own derived copy of a shared setting
- Re-implementing the same setting logic in multiple places

Safe implementation pattern:
1. Define the setting once in shared flag/state definitions.
2. Reuse the same options source anywhere the setting is rendered.
3. Route all changes through one shared setter.
4. Refresh all mirrored controls after state changes.
5. Verify that changing either control updates the other and changes the final command preview.

If a shared control becomes unreliable, prefer removing the duplicate UI over keeping two unsynchronized versions.

## Agent Workflow Guidelines

### Before You Start
- Read the task carefully and identify which files are involved. Use the
  File Ownership Reference in this file to locate the right files.
- Search for existing patterns before writing new code. If a helper, setter,
  or validation function already exists, reuse it.
- Check `docs/todo.md` for known planned work. If your task overlaps with a
  TODO item, follow its acceptance criteria.

### Make Minimal, Focused Changes
- Change only the files necessary for the task. Avoid "while I'm here" edits
  to unrelated code.
- When fixing a bug, fix the root cause. Do not add workarounds that mask the
  symptom (e.g., adding `setTimeout` to hide a race condition).
- When adding a new flag, template, or preset, follow the existing pattern
  exactly. Do not invent a new pattern when one already exists.

### Verify After Every Change
- Run `node --check ui/js/<file>.js` on every JS file you touch to catch
  syntax errors immediately.
- Run `node tests/frontend/custom_launch_args_unit.cjs` after parser changes.
- Run `npm run test:frontend` (Playwright smoke test) after any change to
  mirrored controls, flag state, command preview, or shared setters.
- Run `python -m unittest discover tests -v` after backend changes.
- Test the generated command preview manually if Playwright is not available.

### Prefer Incremental Changes Over Large Refactors
- Decompose large tasks into small, independently verifiable steps.
- Each step should leave the app in a working state.
- Do not batch multiple unrelated changes into one commit or one edit session.

## Common Pitfalls

### Frontend

- **Never mutate `flagValues` directly.** All changes must go through
  `setFlagValue()` / `setMultipleFlagValues()` / `applyFlagValues()` in
  `flag-core.js`. Direct mutation (e.g.,
  `flagCore.getFlagValues().temperature = 0.5`) bypasses the sync broadcast
  and silently breaks Configure, Quick Launch, Chat, and command preview.

- **Never create per-tab copies of shared state.** If a tab needs to display
  a flag value, read from `flagCore.getFlagValues()`. Do not store a local
  copy that can drift out of sync.

- **Never maintain a separate options list for a duplicated setting.** If
  Configure and Quick Launch both show a chat template dropdown, both must
  read from the same `CHAT_TEMPLATE_PRESET_OPTIONS` source in `flags.js`.

- **Do not use `innerHTML` with user/model content.** Use `textContent` for
  user-facing text. The `renderMarkdown()` function is the one exception for
  model output; do not add new `innerHTML` usage.

- **Do not add new global functions to `app.js`.** The file already has 80+
  global functions. New behavior should be namespaced under

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomas9120/LLama-GUI](https://github.com/thomas9120/LLama-GUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
