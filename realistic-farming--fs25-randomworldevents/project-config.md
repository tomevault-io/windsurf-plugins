---
trigger: always_on
description: > **Tyson's ruling, 2026-08-10. BINDING on every human and every agent seat, including Bob, Fred and Sasha, because those seats are the ones opening PRs.**
---

# FS25_RandomWorldEvents - Claude Code Project Instructions

## Git Workflow — ONE FEATURE, ONE PR

> **Tyson's ruling, 2026-08-10. BINDING on every human and every agent seat, including Bob, Fred and Sasha, because those seats are the ones opening PRs.**

- **Every feature, fix or brief gets its OWN BRANCH**, cut fresh from `development`:
  `feat/<ID>-<slug>`, `fix/<ID>-<slug>`, or `docs/<slug>` / `chore/<slug>` for non-code work
  (e.g. `feat/SCS-037-caught-up-hour`).
  Commit **only that one item** on it.
- **The PR is `feat/...` → `development`.** One item per PR, always. Delete the branch on merge.
- **NEVER open a feature PR from `development` itself.** `development` is the trunk: a PR based on it
  silently absorbs every commit that lands while it is open, under a title that still describes the
  first one. **This happened twice in two days**, the second time to the seat that had just reported it.
- **`development` → `main` is a RELEASE PR only**, titled `Release vX.Y.Z`. It may carry many features
  *by design* and its body lists them. It is never a feature PR.
- **Never commit or push directly to `main`.** Check your branch at the start of every session.
- If a PR does end up carrying more than its title says: **retitle, rebody with the full commit list,
  and refresh every approval.** An old approval never covers code it did not see.

```bash
git checkout development && git pull
git checkout -b feat/SCS-037-caught-up-hour
#   ...commit the ONE feature...
git push -u origin feat/SCS-037-caught-up-hour
gh pr create --base development
#   -> Sasha approves -> Tyson merges -> branch deleted
```

**Sasha approves, Tyson merges.** No seat both approves and lands the same PR.

## !! MANDATORY: Before Writing ANY FS25 API Code !!
Before implementing any FS25 Lua API call, class usage, or game system interaction,
ALWAYS check the following local reference folders first. These contain CORRECT,
PROVEN API documentation - they are the ground truth. Do NOT rely on training data
for FS25 API specifics; it may be outdated, wrong, or hallucinated.

### Reference Locations
| Reference | Path | Use for |
|-----------|------|---------|
| FS25-Community-LUADOC | `C:\Users\tison\Desktop\FS25 MODS\FS25-Community-LUADOC` | Class APIs, method signatures, function arguments, return values, inheritance chains |
| FS25-lua-scripting | `C:\Users\tison\Desktop\FS25 MODS\FS25-lua-scripting` | Scripting patterns, working examples, proven integration approaches |

### When to Check (mandatory, not optional)
- Any `g_currentMission.*` call
- Any `g_gui.*` / dialog / GUI system usage
- Any hotspot / map icon API (`MapHotspot`, `PlaceableHotspot`, `IngameMap`, etc.)
- Any `addMapHotspot` / `removeMapHotspot` usage
- Any `Class()` / `isa()` / inheritance pattern
- Any `g_i3DManager` / i3d loading
- Any `g_overlayManager` / `Overlay.new` usage
- Any `g_inputBinding` / action event registration
- Any save/load XML API (`xmlFile:setInt`, `xmlFile:getValue`, etc.)
- Any `MessageType` / `g_messageCenter` subscription
- Any placeable specialization or `g_placeableSystem` usage
- Any finance / economy API call
- Any `Utils.*` helper you are not 100% certain about
- Any new FS25 system not previously used in this project

### How to Check
1. Search the LUADOC for the class or function name
2. Read the full method signature including ALL arguments and return values
3. Check inheritance - many FS25 classes require parent constructor calls
4. Look for working examples in FS25-lua-scripting before writing new code
5. If the API is NOT in either reference, state that clearly rather than guessing

---

## Project Overview

**FS25_RandomWorldEvents** is a Farming Simulator 25 mod (v2.0.0.0) that introduces a
random-event system and a real vehicle-physics layer to the base game. Events fire on a
probabilistic timer during gameplay and can affect the economy, vehicles, fields, animals,
and special game-state variables. All settings persist per-savegame via an XML file.

Author: TisonK | License: All rights reserved.

### Vehicle physics

The vehicle-physics layer (`utils/VehiclePhysics.lua`, the `RWEVehiclePhysics`
specialization) only touches fields the engine actually reads. The earlier code wrote to
non-existent fields (`wheel.physics.frictionScale`, `wheel.suspension.springForce`,
`motor.maxPower`, `vehicle.maxSpeed`) and did nothing - see `VEHICLE_PHYSICS_FINDINGS.txt`
in the repo root for the full wrong-vs-correct writeup, and `DEVELOPMENT.md` §5 for the
architecture.

> **Credit:** the steering technique (writing `spec_drivable.lastInputValues.axisSteer`
> each frame) and the `TypeManager.finalizeTypes` specialization-injection approach are
> adapted from **"RealPhysics Steering" by Tubez47**, recorded inline in the source header.
> Keep that credit intact in any future edits to the physics layer.

---

## Repository Layout

```
FS25_RandomWorldEvents/
├── RandomWorldEvents.lua          # Core manager + FS25 lifecycle hooks
├── modDesc.xml                    # Mod metadata, l10n strings, source file list
├── guiProfiles.xml                # GUI style profiles
├── icon.dds                       # Mod icon
├── icons/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Realistic-Farming/FS25_RandomWorldEvents](https://github.com/Realistic-Farming/FS25_RandomWorldEvents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
