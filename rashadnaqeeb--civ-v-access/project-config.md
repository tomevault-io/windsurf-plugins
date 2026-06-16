---
trigger: always_on
description: Civ-V-Access is an accessibility layer for Sid Meier's Civilization V that makes the game playable for blind users. It reaches into the game through three components: a `lua51_Win32.dll` proxy that binds Tolk as a global `tolk` table inside every Lua context; a fake DLC (deployed to `Assets/DLC/DLC_CivVAccess/` in the game install) that installs UI handlers via `ContextPtr`, `Events.X`, and `LuaEvents.X`; and a forked `CvGameCore_Expansion2.dll` (BNW engine) that exposes additional Lua bindings 
---

# Civ-V-Access — Claude Code Instructions

Civ-V-Access is an accessibility layer for Sid Meier's Civilization V that makes the game playable for blind users. It reaches into the game through three components: a `lua51_Win32.dll` proxy that binds Tolk as a global `tolk` table inside every Lua context; a fake DLC (deployed to `Assets/DLC/DLC_CivVAccess/` in the game install) that installs UI handlers via `ContextPtr`, `Events.X`, and `LuaEvents.X`; and a forked `CvGameCore_Expansion2.dll` (BNW engine) that exposes additional Lua bindings for engine APIs the stock DLL keeps internal. Packaging as a DLC (rather than a mod under `Documents/My Games/.../MODS/`) is what layers our UI files into the engine's Contexts via `<UISkin>` and keeps the session off the mod-hash list for multiplayer. Speech output is the sole interface — there is no visual fallback. Every decision should be weighed against the fact that if something fails silently or speaks stale data, the player has no way to know.

## Changelog

When committing a new feature or bug fix, add an entry to `CHANGELOG.md` under `## [Unreleased]`, beneath one of two section headers: `New Features and improvements:` or `Bug fixes:`. Add the header if it isn't there yet.

Keep entries terse. One sentence per change, from the player's perspective, ideally under ~120 characters. State the change directly and stop.

Player-facing means player language. Spell language names out ("Russian", "Spanish"), not locale codes ("ru", "es"). Use the same names for features, screens, and keys that the user would. The reader is a player reading release notes, not a developer reading a diff.

Do include the hotkey when it's how the player invokes the feature being fixed -- "Reading the unit on a fogged tile" is ambiguous without "with S", because the player has no way to know which action the entry is about. The hotkey is the player-facing surface; omit it only when the change applies regardless of input (passive announcements, layout, behavior on existing screens).

Do not include:

- Pre-fix behavior beyond what the fix itself implies. "Reading the unit on a fogged tile with S no longer leaks the unit hiding there" is the entry; "It used to speak HP and combat strength, but now says no units" is bloat.
- Multiple quoted strings illustrating the same point. At most one short parenthetical example, and only if the entry is ambiguous without it.
- Meta-commentary about parity with sighted players, rationale, or how a sighted player would experience the fix. The reader knows the audience.
- Implementation detail, file paths, internal symbol names.

Earlier release sections in `CHANGELOG.md` contain verbose, padded entries from before this rule. Do not model new entries on them.

## Commit messages

Use the standard subject + body format: a short imperative subject line under ~72 characters, one blank line, then a body wrapped at ~72 characters with paragraphs or bullet lists. Do not merge the subject and body into a single run-on paragraph. Parts of the existing log do this; treat them as antipattern, not template.

## Build

The pipeline is three standalone scripts at the repo root. Each one is run only when its own piece of the codebase has changed; the build outputs are committed, so deploys don't require a prior build.

- `build-proxy.ps1` — compiles `lua51_Win32.dll` from `src/proxy/` to `dist/proxy/lua51_Win32.dll` (committed). Picks the latest Visual Studio install with C++ build tools via `vswhere -latest -products *` and invokes its `cl.exe` through `vcvarsall.bat x86`. Re-run only when `src/proxy/` changes.
- `build-engine.ps1` — compiles the engine DLL fork (`CvGameCore_Expansion2.dll`) from the Civ V Modding SDK source plus our overlay in `src/engine/`, output to `dist/engine/CvGameCore_Expansion2.dll` (committed). Uses VC9 (Visual C++ 2008 SP1) supplied by Windows SDK 7.0's `vc_stdx86` MSI. Engine builds take 1-2 minutes; re-run only when `src/engine/` changes.
- `deploy.ps1` — copies the committed artifacts into the game install: `dist/proxy/lua51_Win32.dll`, the Tolk runtime DLLs from `third_party/tolk/dist/x86/`, the `src/dlc/` payload into `Assets/DLC/DLC_CivVAccess/`, `sounds/*.wav` into the DLC's `Sounds/`, and `dist/engine/CvGameCore_Expansion2.dll`. The engine DLL deploys by default (pass `-SkipEngine` for fast Lua-only iterations); on first install it backs up the vanilla DLL to `build/CvGameCore_Expansion2.vanilla.dll.bak`. `-Uninstall` reverses everything (restoring the vanilla engine DLL if a backup exists).
- `deploy-sighted-multiplayer.ps1` — minimal deploy for a sighted MP partner playing against a Civ-V-Access host. Copies only the engine DLL (GUID match) and `CivVAccess_2.Civ5Pkg` with empty UI directories (DLC-list match), no proxy / Tolk / Lua payload. Run on the partner's machine, not the host's.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rashadnaqeeb/Civ-V-Access](https://github.com/rashadnaqeeb/Civ-V-Access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
