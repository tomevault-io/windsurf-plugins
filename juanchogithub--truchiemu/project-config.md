---
trigger: always_on
description: Behavioral guidelines and project-specific instructions for working on TruchiEmu.
---

# AGENTS.md

Behavioral guidelines and project-specific instructions for working on TruchiEmu.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## Localization method

The app uses a **JSON‑based UI localization system**:
- Translation files in `Resources/Translations/`: `en.json`, `es.json`, `pt.json`
- `LocalizationManager` loads all JSON files at launch, auto-detects device language, supports runtime changes via `setLanguage(_:)`
- **IMPORTANT:** `setLanguage()` persists to `AppSettings.set("systemLanguage", value: lang)`. Without this, language resets on next app launch

### How to use localization in SwiftUI views

**For SwiftUI Section/Picker/Button titles (String parameter):**
```swift
loc.localized("settings.saveStates") // Returns String
Section(loc.localized("settings.saveStates")) { ... }
Picker(loc.localized("settings.selectLanguage"), selection: $binding) { ... }
```

**For SwiftUI Text (SwiftUI Text view):**
```swift
Text("settings.title") // Uses the Text extension, returns localized Text
```

**For confirmation dialog messages:**
```swift
.confirmationDialog(loc.localized("settings.syncAllGamesTitle"), ...) { ... }
```

**Key pattern:** Always use `loc.localized("key")` for String arguments, `Text("key")` for Text arguments.

**Footgun:** The `Text` extension overrides `init(_ localizationKey: String)`, which intercepts ALL `Text(String)` calls, not just localization keys. If you need a literal string in a Text view that isn't a localization key, use `Text(verbatim: "literal")` instead.

### Adding new translations

1. Add key to ALL language JSON files (`en.json`, `es.json`, `pt.json`) with the same key
2. Use consistent naming: `section.action` (e.g., `settings.saveStates`, `game.launch`)
3. Update views to use `loc.localized("key")` instead of hardcoded strings
4. Debug/internal messages are not translated

### Common bug to avoid

When adding a language picker that calls `setLanguage()`:
- Ensure `setLanguage()` saves to `AppSettings`, otherwise the selection is lost when the view re-renders
- The picker binding must read from `loc.currentLanguage` to show the current selection

### Dual language system

The codebase has **two separate language systems**. Do not confuse them:

| System | Class | Storage | Purpose |
|---|---|---|---|
| UI Localization | `LocalizationManager.shared` | `AppSettings["systemLanguage"]` (String: `"en"`, `"es"`, `"pt"`) | UI string translations |
| Core Language | `SystemPreferences.systemLanguage` | `AppSettings` (Int: `EmulatorLanguage` raw value) | Language passed to libretro cores via `LibretroBridge.setLanguage(Int32)` |

Changing one does NOT affect the other. `LocalizationManager.setLanguage()` posts `Notification.Name.languageChanged` (defined in `TruchiEmuApp.swift`, not in `LocalizationManager`).

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

# TruchiEmu Developer Guide

## Build System

- **XcodeGen**: Run `xcodegen generate` after any `project.yml` change to regenerate `TruchiEmu.xcodeproj`. Do not edit the `.xcodeproj` directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuanchoGithub/truchiemu](https://github.com/JuanchoGithub/truchiemu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
