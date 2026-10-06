---
trigger: always_on
description: A curated collection of World of Warcraft addons for the **3.3.5a client** (interface version `30300`, Lua 5.1). Several addons have been back-ported from newer WoW versions and modified for compatibility. The language is Lua; no build toolchain, tests, or linter exist.
---

# GitHub Copilot Instructions

## Repository Overview

A curated collection of World of Warcraft addons for the **3.3.5a client** (interface version `30300`, Lua 5.1). Several addons have been back-ported from newer WoW versions and modified for compatibility. The language is Lua; no build toolchain, tests, or linter exist.

---

## Release Workflow

Releases are automated via `.github/workflows/release-addon.yml`. To release an addon:

1. Push a tag in the form `AddonName-release` (e.g. `HidingBar-release`).
2. The workflow reads `## Version:` from `AddonName/AddonName.toc`, packages all folders whose name starts with `AddonName` (e.g. `HidingBar` + `HidingBar_Options`), and creates a GitHub Release with a zip attachment.

The `## Version:` field in the `.toc` file is the single source of truth for the release version.

> **Back-port rule:** When fixing or modifying an existing back-ported addon, do **not** bump `## Version:` in the `.toc` file. Version bumps only happen as part of an intentional release.

---

## TOC File Conventions

Every addon's `.toc` must include:

```
## Interface: 30300
## Version: X.Y.Z
```

- `30300` is the only valid interface number for this client.
- Back-ported addons load `compat_335.lua` (or a named equivalent) as the **first file** in the TOC so all shims are in place before any addon code runs.

---

## Back-porting Addons to 3.3.5a

When backporting an addon from a newer WoW version, the pattern used in this repo is:

1. **Change `## Interface:`** to `30300`.
2. **Create `compat_335.lua`** in the addon folder — a self-contained shim file loaded first by the TOC.
3. **Wrap every shim with `if not X then … end`** so it never overwrites a real API the server already provides.
4. **Document every change** in a `README.md` in the addon folder, including which APIs were shimmed and why.

### APIs absent from 3.3.5a that frequently need shimming

| API / namespace | What to do |
|---|---|
| `C_Timer.After` / `NewTimer` / `NewTicker` | Implement via a Frame `OnUpdate` driver |
| `C_Container.*` | Use legacy globals (`GetItemCooldown`, etc.) directly |
| `C_ChatInfo.SendAddonMessage` | Use `SendAddonMessage()` global |
| `C_Item.IsEquippedItem` | Stub `C_Item = {}` delegating to the global |
| `C_UnitAuras.GetPlayerAuraBySpellID` | Iterate `UnitAura("player",…)` matching by spell name via `GetSpellInfo` |
| `C_AddOns.*` | Alias legacy globals (`GetAddOnMetadata`, `IsAddOnLoaded`, etc.) |
| `C_Texture.GetAtlasInfo` | Stub returning `nil` |
| `securecallfunction` | `function(func, ...) return func(...) end` |
| `GetPhysicalScreenSize` | `return GetScreenWidth(), GetScreenHeight()` |
| `BackdropTemplateMixin` / `CreateFromMixins` | Stub empty table / shallow-copy helper |
| `LOADING_SCREEN_DISABLED` event | Use `PLAYER_LOGIN` instead |
| `RegisterAddonMessagePrefix` | Remove the call (doesn't exist in 3.3.5a) |
| `WOW_PROJECT_*` constants | Define manually; set `WOW_PROJECT_ID = WOW_PROJECT_WRATH_CLASSIC` |
| `CombatLogGetCurrentEventInfo` | Register a frame for `COMBAT_LOG_EVENT_UNFILTERED`, store args, expose via a wrapper |
| Settings framework (`Settings.RegisterCanvasLayoutCategory`, etc.) | Use `InterfaceOptions_AddCategory` / `InterfaceOptionsFrame_OpenToCategory` |
| `SOUNDKIT` table | Map the needed sound IDs to their raw numeric values |
| `HybridScrollFrame_*` | Re-implement on top of `FauxScrollFrame` semantics |

### Frame / widget method shims

These methods are missing from the 3.3.5a widget API and should be shimmed onto the frame metatable's `__index` **only when it is a Lua table** — in 3.3.5a `getmetatable(frame).__index` is a **C dispatcher function**, not a Lua table, so you cannot inject methods this way at startup. Patch onto individual frame instances directly, or use `ADDON_LOADED`/`PLAYER_LOGIN` timing:

| Method | Polyfill |
|---|---|
| `frame:SetShown(bool)` | `if bool then self:Show() else self:Hide() end` |
| `frame:SetColorTexture(r,g,b,a)` | `SetTexture` + `SetVertexColor` |
| `frame:SetClipsChildren(...)` | no-op / guard with `if btn.SetClipsChildren then` |
| `frame:SetFixedFrameStrata` / `SetFixedFrameLevel` | no-op / guard with existence check |
| `frame:IsMouseMotionEnabled` / `SetMouseMotionEnabled` | delegate to `IsMouseEnabled` / `EnableMouse` |
| `anim:SetFromAlpha` / `SetToAlpha` / `SetStartDelay` | no-op (animation fading unsupported) |
| `animGroup:SetToFinalAlpha` | no-op |
| `anim:GetTarget` / `GetObjectType` | guard: `animation.GetTarget and animation:GetTarget()` |

### Lua 5.1 limitations

- `xpcall(f, handler, arg1, ...)` — extra arguments after the handler are **silently ignored**. Wrap in a closure: `xpcall(function() f(arg1) end, handler)`.
- `getmetatable(frame).__index` is a C function — indexing it for method injection returns `nil`. Use the "nil-the-hook, call native, re-hook" pattern:
  ```lua
  local function MyHook(self, ...)
      self.MethodName = nil
      local result = self:MethodName(...)
      self.MethodName = MyHook
      return result
  end
  ```

### Textures


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [locus313/WoW-3.3.5a-Addons](https://github.com/locus313/WoW-3.3.5a-Addons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
