---
trigger: always_on
description: - Never run `npm/pnpm format` or `npm/pnpm bf`
---

# Yuki OS - Agent Reference

## Hard Rules

- Never run `npm/pnpm format` or `npm/pnpm bf`
- Never add comments, docstrings, or `/* */` blocks in CSS, JS, or HTML other than "JSDoc" for complex functions
- Never spawn a browser for testing
- Before finalizing any code changes, run `pnpm build:dev` in `webos-desktop/`. A change that breaks the build is
  incomplete.
- Always use CSS variables from `src/styles/style.css`. Never hardcode colors.
- When making significant changes, new features, or new apps: you must register them in src/news.js with an icon, title, and a
  punchy, active-voice description under 15 words. Bad: 'First-time setup now includes a dedicated profile step...'
  Good: 'Choose your nickname and avatar during setup, with a quick final preview!'
- Whenever you define a new app to appJauncher or gamesJist, define description for it on gameDescriptions.js
- Always use StorageKeys from `src/StorageKeys.js` for localStorage access. Never hardcode localStorage key strings.
  Import StorageKeys and use the defined constants. If a new key is needed, add it to StorageKeys.js first.
- Always use `os.storage` API instead of bare `localStorage`; the storage module handles serialization/deserialization automatically.
- Never use browser native alerts, prompts, or confirms (alert(), prompt(), confirm()). Always use the shared
  dialog utilities from `src/shared/dialogs.js` instead. Import and use `showAlert`, `showPrompt`, `showConfirm`,
  `customAlert`, `customPrompt`, or `customConfirm` as appropriate.
- Never use `document.querySelector`, `document.querySelectorAll`, or direct DOM manipulation methods. Always use the
  utility functions from `src/shared/domUtils.js` instead. Import and use `$` (querySelector), `$$` (querySelectorAll),
  `bindEvent`, `toggleClass`, `setText`, `setHTML`, `createElement`, etc. For general utility functions, use `src/utils/utils.js`
  (e.g., `formatSize`, `isImageFile`, `isTextFile`, `pluralize`).
- Use os.notify.send() for discrete, user-facing application events that represent a state change or completion, and ensure notifications are not emitted from high-frequency, repeating, or continuously-updating processes.
- If a change introduces a new system, abstraction, manager, API surface, or reusable capability, create a new file and integrate it via imports. Only modify existing files if the change is a direct refinement of existing logic without introducing a new responsibility boundary.
---

## Code Quality Guidelines

Write modular, clean, and DRY code. Follow these principles:

- **Modularity**: Separate concerns into focused modules. Each file should have a single, clear responsibility. Avoid
  monolithic files that handle multiple unrelated concerns.
- **Single Responsibility**: Functions and classes should do one thing well. If a function does more than one thing,
  split it into smaller, focused functions.
- **DRY (Don't Repeat Yourself)**: Never duplicate logic. Use shared utilities from `src/shared/` instead of
  reimplementing common functionality. If you find yourself writing the same code in multiple places, extract it into a
  reusable function.
- **Prefer Existing Utilities**: Before writing new utility functions, check `src/shared/` for existing helpers. Common
  patterns like dialogs, asset resolution, and platform detection already have implementations.
- **Clean Function Names**: Use descriptive, action-oriented function names. `installApp()` is better than `doIt()`.
  `validateUrl()` is better than `check()`.
- **Avoid Deep Nesting**: More than 3 levels of nesting indicates a need for refactoring. Use early returns and guard
  clauses to reduce nesting.
- **Keep Functions Small**: Functions should fit on a screen (typically < 50 lines). If a function is longer, consider
  splitting it into smaller helper functions.
- **Use Meaningful Variables**: Variable names should reveal intent. `userList` is better than `data`. `isValid` is
  better than `flag`.
- **Avoid Magic Numbers/Strings**: Extract constants to the top of the file or a constants file. Use CSS variables for
  styling values.
- **Consistent Patterns**: Follow existing patterns in the codebase. If similar apps use a certain structure, follow that structure for new apps.
- **Enforce KISS and YAGNI:** Write the absolute minimum code required to make current tests pass; do not build abstract factories, extra interfaces, or future-proof scaffolding for features that are not explicitly requested in the prompt.

---

## Styling System

Yuki OS uses a dark glassmorphism theme with a comprehensive theming system. All rules below are non-negotiable.

- **CSS Variables**: Use `--brand` (accent), `--text-primary`, `--text-secondary`, `--bg-primary`, `--bg-secondary`,
  `--glass`, `--glass-border`, `--error`. Never introduce new hues or hardcoded values.
- **Color Hue**: All colors use unified hue 265 (purple). Never mix in gray or blue hues.
- **Glassmorphism**: `backdrop-filter: blur(32px+)`, semi-transparent `rgba` backgrounds (0.6–0.98 opacity), subtle
  borders (`rgba(255,255,255,0.08–0.12)`).
- **Depth**: Multi-layer box shadows - `0 24px 64px rgba(0,0,0,0.65)` + inset highlight.
- **Typography**: System fonts or JetBrains Mono for code. 13–16px base (minimum 12px for any readable text). Opacity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Reeyuki/YukiOS](https://github.com/Reeyuki/YukiOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
