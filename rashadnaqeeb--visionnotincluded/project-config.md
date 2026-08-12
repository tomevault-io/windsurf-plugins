---
trigger: always_on
description: OniAccess is an accessibility mod for Oxygen Not Included that makes the game playable for blind users. It uses Harmony patches to hook into the game's UI and provides speech output as the sole interface — there is no visual fallback. Every decision should be weighed against the fact that if something fails silently or speaks stale data, the player has no way to know.
---

# OniAccess - Claude Code Instructions

OniAccess is an accessibility mod for Oxygen Not Included that makes the game playable for blind users. It uses Harmony patches to hook into the game's UI and provides speech output as the sole interface — there is no visual fallback. Every decision should be weighed against the fact that if something fails silently or speaks stale data, the player has no way to know.

## Build

Always use the build script, never `dotnet build` directly. 

```
powershell -ExecutionPolicy Bypass -File build.ps1
```

The script builds the DLL, deploys it to the game's local mods directory, and patches mods.json to keep the mod enabled.

When a build fails on a type or method signature, look it up in `ONI-Decompiled/` before guessing at fixes.

## Project Structure

- `OniAccess/` - mod source code (C#, .NET Framework 4.8, Harmony patches)
- `ONI-Decompiled/` - decompiled game source for reference (read-only, not part of build)
- `docs/` - design documentation
- `docs/game-mechanics/` - game mechanics reference (topic files, wiki articles, strategy guides). See its CLAUDE.md for details.
- `.planning/` - project planning files
- `changes.md` - changelog for user-facing features and bug fixes

## Changelog

When committing a new feature or bug fix, add an entry to `changes.md`. Keep entries short — one line per change, written from the player's perspective (what changed for them, not implementation details).

## Code Style

- Harmony patch classes: `GameType_MethodName_Patch` (e.g., `KScreen_Activate_Patch`)
- All speech goes through `SpeechPipeline`, never call `SpeechEngine.Say()` directly
- All logging goes through `Log.Info/Debug/Warn/Error`, never use `Debug.Log` directly

## Test

```
powershell -ExecutionPolicy Bypass -File test.ps1
```

Builds and runs the offline test suite (`OniAccess.Tests`). Tests run without the game. All new tests must work offline — never add tests that require launching the game. Don't test individual screen handlers.

- Every test should have a plausible failure mode not covered by another test — don't test the same invariant twice
- Always test real code paths; never test local helpers that simulate production behavior
- Exception: TextFilter-style regression suites keep full coverage (chain of replacements where any change can break unrelated cases)
- Guard speech-boundary code even when it looks simple — a wrong value reaching the speech engine is a silent failure

## Project Rules

### Reuse game data, avoid hardcoding
Use the game's localized text (`STRINGS` namespace, `LocText` components), UI state, and entity data wherever possible. Hardcoded text becomes stale across game updates and blocks translation. Only hardcode when no game data source exists.

### Never cache game state
Do not copy game data into mod-side dictionaries, lists, or string fields for later use. Always re-query the game when you need a value. A sighted player can see when the screen contradicts itself; a blind player trusts speech absolutely. Stale data is worse than no data. The only acceptable "cache" is holding a reference to a live Unity component (e.g., a `KSlider` or `LocText`) and reading its properties at speech time.

### No inline non-punctuation string literals
All user-facing text must come from a `LocString` reference. Never inline string literals for text that gets spoken. Prefer the game's `STRINGS` namespace — search `ONI-Decompiled/` for existing localized text before adding to `OniAccessStrings.cs`. The game already has strings for common labels ("Embark", "Close", "Cancel", etc.). Only add mod-authored strings when no game equivalent exists.

### Concise announcements
**These rules apply to mod-authored text only; never alter, truncate, or reword game text.** Users are experienced screen reader users. Strip fluff, never strip information.
- No positional item counts ("3 of 10") — the screen reader already tracks list position
- No navigation hints ("press Enter to select") unless unusual controls, and on a delay
- No redundant context ("You are now in...")
- No type suffixes when obvious ("Lumber button")
- DO include all gameplay-relevant details (traits, difficulty, descriptions). Concise means no fluff, not less information
- The sooner a message's varying part appears, the faster the user can keep going. Put the distinguishing word first.
  - WRONG: "cursor anchored" / "cursor unanchored" - user must listen through "cursor" before hearing the difference.
  - CORRECT: "anchored cursor" / "unanchored cursor" - first syllable already differs.
- Avoid emdash. Screen readers announce it as "dash" which breaks the flow of speech

### Conscious hotkey management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rashadnaqeeb/VisionNotIncluded](https://github.com/rashadnaqeeb/VisionNotIncluded) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
