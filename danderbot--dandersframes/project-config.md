---
trigger: always_on
description: DandersFrames is a custom party and raid frame addon for World of Warcraft (Retail/Midnight 12.0). It replaces the default Blizzard party/raid frames with highly configurable alternatives. The addon includes a full GUI settings panel, profile system, click-casting system, test mode, aura filtering, and an external API for Wago UI Pack integration.
---

# DandersFrames - Claude Code Project Guide

## Overview

DandersFrames is a custom party and raid frame addon for World of Warcraft (Retail/Midnight 12.0). It replaces the default Blizzard party/raid frames with highly configurable alternatives. The addon includes a full GUI settings panel, profile system, click-casting system, test mode, aura filtering, and an external API for Wago UI Pack integration.

**Author:** Danders
**Current Version:** 4.0.6
**Language:** Lua + XML (WoW addon)
**Slash Commands:** `/df` (settings), `/rl` (reload UI), `/dfarena` (test arena mode)

## Important Workflow Rules

- **Always present a plan before implementing major features or changes.** Outline which files will be modified, what will be added, and how it will work. Wait for approval before writing any code.
- **For small bug fixes or minor tweaks, just go ahead** — no plan needed.
- **Never modify files in the `Libs/` folder.** These are third-party libraries.
- **Always update the TOC file** (`DandersFrames.toc`) when adding or removing Lua/XML files. Load order matters — see `.claude/docs/project-structure.md`.
- **Test mode exists** — when suggesting testing, remind that `/df test` or the test mode toggle can be used to simulate party/raid frames without being in a group.
- **No AI attribution in commits.** Never add `Co-Authored-By` lines, Claude references, or any AI-related mentions in commit messages or code comments.
- **`CHANGELOG.md` is the single source of truth for changelogs.** Always add new changelog entries to `CHANGELOG.md` — never edit `Changelog.lua` directly. CI generates `Changelog.lua` from `CHANGELOG.md` at build time via `generate_changelog.sh`. To update the local in-game changelog for testing, run `bash generate_changelog.sh` then restore side effects with `git checkout -- CHANGELOG.md DandersFrames.toc` (the script modifies these as a side effect). The `/bump-version` skill should add entries to `CHANGELOG.md`.
- **Changelog style: short, sweet, non-technical.** Entries are read by end users in-game. Keep them brief and describe WHAT changed, not HOW.
    - **Default length:** one sentence, ideally under ~20 words. Phrase as the user-visible outcome (what they'll notice).
    - **Never include:** file names, line numbers, function names, API names, event names, secure-template internals, attribute strings, code snippets, commit-message-style explanations of the implementation, or references to how other addons solve the same problem. Don't mention other addons (ElvUI, Grid2, Blizzard internal templates, etc.) unless the user specifically asks.
    - **Allowed to be longer and more detailed:**
        - **New features** — describe what users can do with the feature and where to find the setting. Two to four sentences is fine.
        - **Developer-facing changes (DF API)** — external integrations (e.g. `RegisterCallback` events, public API functions other addons depend on) need enough detail for developers to update their code. This is essentially the only case where technical detail belongs.
    - **Examples:**
        - ❌ "Fix `attempt to perform boolean test on field 'dfInRange' (a secret boolean value, while execution tainted by 'DandersFrames')` error spam from Range.lua:550."
        - ✓ "(Range) Fix error spam when range fading is active in combat."
        - ❌ "Boss frames now register their own unit events directly (`UNIT_HEALTH`, `UNIT_POWER_UPDATE`, `UNIT_AURA`, …). This follows the same pattern ElvUI uses for its boss frames."
        - ✓ "(Friendly Boss NPC Frames) Fix health, power, name, and absorb updates not applying reliably."
- **Open bug awareness.** `_Reference/open-bugs.json` is a local cache of all currently triaged confirmed bugs (regenerated daily after the triage agent, or on demand via `/refresh-bugs`). When implementing fixes or features, you may optionally invoke `/check-bugs` to see if your changes touch files listed in open bug reports — useful for catching "this incidentally fixes bug X" moments or spotting bugs close to what you're working on. The release skills (`/release-alpha`, `/release-beta` manual path, `/release-stable`) run `/check-bugs` automatically before tagging so nothing ships without considering whether open bugs are now resolved.

## File Paths

### Repository (source of truth)
```
C:\Users\luked\Documents\DandersFrames\Repo\DandersFrames
```

### WoW AddOns Folders (symlinked to repo)
```
Live:  C:\Games\Blizzard\World of Warcraft\_retail_\Interface\AddOns\DandersFrames
PTR:   C:\Games\Blizzard\World of Warcraft\_ptr_\Interface\AddOns\DandersFrames
Beta:  C:\Games\Blizzard\World of Warcraft\_beta_\Interface\AddOns\DandersFrames
```
All three are symlinked to the repo folder — changes in the repo are immediately available in all WoW installations.

### Reference Folder
```
_Reference/
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanderBot/DandersFrames](https://github.com/DanderBot/DandersFrames) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
