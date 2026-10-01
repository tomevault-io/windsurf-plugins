---
trigger: always_on
description: Cyberpunk 2077 mod: flight for all vehicles. RED4ext C++ plugin + redscript + TweakXL tweaks + ArchiveXL archive + FMOD audio banks. Everything ships under `red4ext/plugins/let_there_be_flight/` (scripts, tweaks, archive, inputs are all loaded from the plugin folder, not r6/archive dirs).
---

# Let There Be Flight (LTBF)

Cyberpunk 2077 mod: flight for all vehicles. RED4ext C++ plugin + redscript + TweakXL tweaks + ArchiveXL archive + FMOD audio banks. Everything ships under `red4ext/plugins/let_there_be_flight/` (scripts, tweaks, archive, inputs are all loaded from the plugin folder, not r6/archive dirs).

Sibling repos with the same build/release shape and their own CLAUDE.md: `../mod_settings` (Nexus 4885), `../in_world_navigation` (Nexus 4583), `../input_loader` (Nexus 4575). SDK/cyberpunk_cmake updates are done here first and the commits reused there.

## Layout

- `src/red4ext/` - the plugin. `Main.cpp` = entry, dependency version checks, registers scripts/tweaks/archive/inputs with the other plugins. `Hooks/` = one game-function hook per file. `Utils/FlightModule.hpp` = hook registry macros.
- `src/redscript/` - 71 `.reds` files, packed by CMake into `packed.reds` + `module.reds`.
- `src/tweaks/` - `.tweak` (native) + `.yaml` tweaks, packed into one file each.
- `src/wolvenkit/` - Wolvenkit project. The built `packed/archive/pc/mod/let_there_be_flight.archive` is committed; CMake just copies it. Rebuild in Wolvenkit only when the archive content changes.
- `src/fmod_studio/` - FMOD project. Built banks `Build/Desktop/*.bank` are committed; CMake copies them plus `fmod.dll`/`fmodstudio.dll`.
- `src/input_loader/let_there_be_flight.xml` - keybinds, installed as `inputs.xml`.
- `deps/` - git submodules (see below). `deps/fmod`, `deps/PhysX_3.4`, `deps/PxShared` are NOT submodules (gitignored local SDK drops) - do not delete.
- `deps/cyberpunk_cmake/` - our CMake framework (`configure_mod`, `configure_red4ext`, `configure_redscript`, `configure_tweaks`, `configure_release`, `configure_install`). Ships `tools/` (zoltan-clang, redscript-cli, ninja).
- `experiments/`, `examples/` - scratch, not built.
- `game_dir/` - build output in game-folder layout. `game_dir_debug/` - PDBs. Both zipped for release.

## Build

Local toolchain: MSVC 2022 (VS Community, kit in `.vscode/cmake-kits.json`), Ninja, CMake 3.24+. Configure and build:

```
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -G Ninja
cmake --build build
cmake --install build      # copies game_dir + game_dir_debug into CYBERPUNK_2077_GAME_DIR
cmake --build build --target let_there_be_flight_release   # zips game_dir -> let_there_be_flight_<ver>.zip
```

- Game dir is read from the `CYBERPUNK_2077_GAME_DIR` cache var (currently the Steam install). Game version for the `.rc`/badge comes from `Cyberpunk2077.exe` FileVersion via `deps/red4ext.sdk/cmake/GetGameVersion.cmake`; in CI there is no exe so it falls back to the SDK's `RED4EXT_RUNTIME_LATEST` macro. Keep the fork's `Api/v0/Runtime.hpp` `RUNTIME_LATEST` pointing at the current patch or the release name/badge will be wrong.
- Mod version comes from the latest git tag (`ConfigureVersionFromGit`). Untagged local builds get `+<branch>.<n>.<sha>` metadata (CI builds drop it).
- `game_dir_requirements/` is NOT safe to copy wholesale into the game: `FindRED4ext`/`FindArchiveXL`/`FindTweakXL` unpack whatever release zip is cached in `build/downloads/` (fetched once, then never refreshed unless `-DMOD_FORCE_UPDATE_DEPS=ON`), so it can hold years-old RED4ext/ArchiveXL/TweakXL. Do not use it for local testing at all: the in-tree `input_loader.dll` / `mod_settings.dll` report wrong plugin versions (input_loader 0.1.1 from its hardcoded project version, mod_settings the parent's LTBF version), so `Main.cpp`'s dependency check rejects them. Install the real Input Loader / Mod Settings release zips into the game instead.
- `configure_release` regenerates `requirements.md` from `MOD_REQUIREMENTS` (own entries in root `CMakeLists.txt`, plus whatever ArchiveXL/TweakXL/RED4ext Find modules append). It is committed and appended to release notes.
- Redscript is compiled/linted against `deps/mod_settings/redscript` too (`.redscript-ide`).
- Dead config, safe to ignore: `src/red4ext/CMakeLists.txt` and `cmake/FindCodeware.cmake` are never included (the plugin target is built by `configure_red4ext` globbing the dir); the `src/redscript/codeware` submodule in `.gitmodules` is not checked out and not needed. Nothing in the plugin includes Codeware headers.
- `ZOLTAN_CLANG_EXE` is hardcoded in root `CMakeLists.txt` to a local zoltan build; `configure_red4ext_addresses` is commented out, so zoltan does not run in the normal build.

## How the plugin finds game functions (what a game patch actually breaks)

- Almost every hook is `REGISTER_FLIGHT_HOOK_HASH(ret, <hash>, Name, args...)` in `src/red4ext/Hooks/*.cpp`. The hash is a RED4ext universal address hash resolved at runtime by `RED4ext::UniversalRelocBase::Resolve`, i.e. by the installed RED4ext's address database for the running game version. `UniversalRelocFunc<...>(hash)` is used the same way for calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jackhumbert/let_there_be_flight](https://github.com/jackhumbert/let_there_be_flight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
