---
trigger: always_on
description: Non-Visual Calculus (code identifier `NonVisualCalculus`, named after the game's Visual Calculus skill)
---

# Non-Visual Calculus - Claude Code Instructions

Non-Visual Calculus (code identifier `NonVisualCalculus`, named after the game's Visual Calculus skill)
makes **Disco Elysium (The Final Cut)** playable by blind users. Speech is the sole
interface, so if something fails silently, speaks stale data, or omits information, the player has no
way to know. A logged failure is actionable; a silent one is invisible.

The game is a dialogue-heavy isometric RPG with no combat and no real-time action, so the work splits
the way our reference mods did: a **UI layer** (menus, dialogue, inventory, thought cabinet, character
sheet) read by hooking the game's focus system and announcing, and a **world layer** (the isometric
scene you move through and click) read with a cursor plus a scanner of interactables. We ride the
game's own controller-navigation and pathfinding.

## Game & environment

- Engine: **Unity 2020.3.12f1, IL2CPP, x64** (build 2022-05-20, ZAUM). Game code is native
  (`GameAssembly.dll` + `il2cpp_data/Metadata/global-metadata.dat`), so we work through Il2CppInterop proxies.
- Install: the build resolves the game folder itself (`GameDir`), so no machine path is committed - it
  reads Steam's main library from the registry, with overrides via `-p:GameDir=...`, the
  `DISCO_ELYSIUM_DIR` env var, or a gitignored `Directory.Build.local.props` (see `Directory.Build.props`).
  For an install outside Steam's main library, set one of those overrides. The game must
  be launched **through Steam**; running `disco.exe` directly exits on the DRM check.
- Loader: **BepInEx 6.0.0-pre.2, Unity.IL2CPP win-x64** (vendored in `third_party/bepinex/`). It runs
  Il2CppInterop on first launch to generate managed proxy assemblies under `<game>\BepInEx\interop\` -
  those are our compile targets (Il2Cpp-prefixed types). Bundles HarmonyX. Mod assemblies run on
  **.NET 6** (BepInEx's bundled CoreCLR), so the plugin targets `net6.0`.
- Middleware: UI is **uGUI + TextMeshPro**; dialogue is the **Pixel Crushers Dialogue System**
  (`DialogueSystem.dll`, `PixelCrushers.dll`); localization is **I2** (`l2Localization.dll`). DE wraps
  much of its UI in a custom "Sunshine" / "Pages" framework (`PagesSystem`, `Pages.Gameplay.*`, `SubPage`).
- Speech backend is **Prism** (https://github.com/ethindp/prism), bound via hand-written P/Invoke
  against `prism.dll`, vendored in `third_party/prism/` and deployed next to `disco.exe`.
- Logs: our lines go through BepInEx logging with a `[Non-Visual Calculus]` prefix into
  `<game>\BepInEx\LogOutput.log` (truncated each launch).

## Decompiled reference

`decompiled/` (gitignored) holds the game's class surface for lookup: `dummydll/` (Cpp2IL stub DLLs)
and `src/` (ilspy C# per type - Assembly-CSharp, DialogueSystem, PixelCrushers, l2Localization). Look
up any game type/method/field signature here before guessing.

**Caveat: the Cpp2IL dummy DLLs have accurate signatures but EMPTY method bodies** (everything returns
`false`/`default`/`null`). Read structure from them, never logic. For real behavior, prefer the Ghidra
pipeline (next paragraph); failing that, re-dump targeted classes with Cpp2IL's IL-recovery mode
(`tools/Cpp2IL.exe`), read the public Pixel Crushers docs, or confirm live with debug logging.

**Real method bodies via Ghidra (`tools/re/`).** The game binary is decompiled to readable C with
il2cpp method names, named struct fields (`this->fields.textOverride`), typed parameters, call
targets, and string literals resolved to their text. **Prefer reading this over probing the live game
through the dev HTTP server** when working out how a method behaves, where a hook belongs, or what text
a call returns: the decompiled code is ground truth read in one shot, whereas a live probe only samples
one hypothesis at a time and leaves uncertainty. Use the HTTP server to confirm runtime values, not to
discover structure. Two artifacts: the full pre-decompiled tree at `decompiled/ghidra/` (one `.c` per
type under namespace dirs, plus `_strings.txt`) to browse and grep, and the saved Ghidra project for
fresh single-class dumps via `tools/re/decompile.sh 'Type$$'` (the `$$` separates Type from Method,
e.g. `SenseOrb$$`), which writes `decompiled/ghidra/<query>.c` with `$` replaced by `_`
(`SenseOrb__.c`). After a game update run `tools/re/refresh.sh` once to rebuild everything. See
`tools/re/README.md`.

**Accessibility in `decompiled/` is not the proxy's.** The decompiled sources report each method's
original accessibility, but the Il2CppInterop proxies we actually compile against
(`<game>/BepInEx/interop/`) generate most members public. A method shown `private` in `decompiled/`
(e.g. `QuicktravelController.IsQuicktravelAvailable()`) is usually still callable from
`NonVisualCalculus.Module` and the dev REPL, so don't conclude "private, won't compile" from the source;
verify by building (`dotnet build`) or calling it in the REPL. To check a proxy method is bound to
native rather than returning a stub default, call a sibling whose true value differs from the default
(`IsOutside()` returning true proved the binding works).

## Build & deploy

`dotnet build` is the whole loop. The `NonVisualCalculus` project has a post-build target (Debug only) that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rashadnaqeeb/NonVisualCalculus](https://github.com/rashadnaqeeb/NonVisualCalculus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
