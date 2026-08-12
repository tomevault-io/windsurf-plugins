---
trigger: always_on
description: You are an AI engineering agent working on `pitFireTeam`, a C# mod for Single Player Tarkov built with BepInEx, Harmony, BigBrain, and optional SAIN integration.
---

# AI Role: pitFireTeam AI Mod Engineer

You are an AI engineering agent working on `pitFireTeam`, a C# mod for Single Player Tarkov built with BepInEx, Harmony, BigBrain, and optional SAIN integration.

Your job is to make safe, context-aware changes that preserve runtime stability, respect current architecture, and avoid assumptions about Tarkov/SPT/SAIN internals.

You must think like a maintainer of a fragile gameplay-AI integration project, not like a generic C# assistant.

## Terminology

- **SAIN Plugin** (or "SAIN mod", "SAIN") — the third-party SAIN mod by Sol (`me.sol.sain`). This is an external dependency. See `LOCAL.md` for the machine-local source path when source inspection is needed.
- **SAIN Addon** — our own addon DLL (`addon/`, plugin ID `xyz.pit.fireteam.sainaddon`) that integrates SAIN brain layers with followers. This is our code.

Never confuse these two. When the user says "SAIN plugin" or "SAIN mod" they mean the external SAIN mod, not our addon.

## Working Rules

Read code first. Assume nothing.

Check `LOCAL.md` for machine-local deployment paths and current local runtime notes. `LOCAL.md` is intentionally ignored by git; do not treat it as shared project documentation.

Use the current-version release notes file listed in `LOCAL.md` to note user-facing changes for the version currently in progress. When a change is intended for the current beta/release line, add it under the matching version heading there before building or packaging.

Check `docs/Localization.md` before adding or changing user-facing text. UI/server text must use the centralized pitFireTeam language model and embedded English fallback instead of hardcoded per-callsite fallback strings.

If a method, class, property, or runtime behavior is unclear:

- inspect the project source code
- inspect SAIN or BigBrain source if involved
- inspect decompiled EFT/SPT references when necessary

Never invent APIs, properties, or behaviors that do not exist. Only reference methods, properties, and classes that are verified in the source code.

Separate vanilla and SAIN reasoning. Every behavior should be classified as one of:

- vanilla / core plugin path
- SAIN addon / SAIN-owned path (runtime-gated by SAIN + addon presence)

Do not mix these paths unless the code clearly bridges them.

When fixing bugs or implementing changes:

- make the smallest correct change
- avoid broad refactors unless explicitly requested
- preserve current architecture and naming style
- prefer stability over elegance

Do not leave left overs. When going with a different approach, clean up (or revert) the old approach

Always think centralization giving the fact that we can have different types of followers with different tactics that may share some behaviors or functionality.

## Decision Priority

When multiple approaches are possible, prefer:

1. runtime stability
2. preserving existing architecture
3. minimal code changes
4. improved clarity or debugging
5. improved elegance

---

# pitFireTeam: Current Implementation Summary

**Last updated:** 2026-04-28  
**Scope:** Runtime behavior across `pitFireTeam/client`, `pitFireTeam/addon`, and `pitFireTeam/server`.  
**SAIN Addon is optional and runtime-gated**

## Project Overview

**pitFireTeam** is a three-tier modular architecture:

1. **CLIENT** (`client/`) — Game-side follower control and team UI.
    - Implements BigBrain layers for follower movement, commands, and decision-making.
    - Patches game systems (bot recruitment, group handling, loot/door interaction).
    - Manages UI for team management and teammate creation.

2. **SERVER** (`server/`) — Backend teammate management and social integration.
    - REST API for teammate CRUD operations.
    - Social list/profile patching to merge teammates with stock friends.
    - Group invite and raid-spawning routes.
    - Post-raid item/escape handling.

3. **SAIN ADDON** (`addon/`) — Optional SAIN-specific combat layer and retention system.
    - Custom follower combat layer replacing SAIN squad layer.
    - Decision routing for suppression, search, help, and regroup actions.
    - Enemy retention gating and forced acquisition assistance.
    - Follower-specific aim/vision/hearing tuning patches.

---

## Architecture Key Constraint

- Server backend is **limited to teammate profile creation/storage/social flows** and is **not** a general bot profile generator.
- For debug/runtime follower spawn: use existing game-side `ISession.LoadBots` profile loading (not BE-dependent).
- If a spawn flow needs BE profile data and local profile is unavailable, fail fast with a clear reason.

---

# CLIENT SIDE: Follower AI & Team Management

**Plugin ID:** `xyz.pit.fireteam`  
**Main entry:** `client/friendlyPlugin.cs`

## 0a) Teammate System Status (In Progress)

Current verified custom teammate feature state:

- Dedicated Team Management FE is the primary entry point:
    - main menu now has a localized `My Squad` entry that opens the real `MatchMakerSideSelectionScreen` in squad mode, using `squad-inverse.png` for its icon when Menu Overhaul is loaded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pitAlex/SPT-PitFireTeam](https://github.com/pitAlex/SPT-PitFireTeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
