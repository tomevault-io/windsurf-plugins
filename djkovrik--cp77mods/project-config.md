---
trigger: always_on
description: These instructions apply to all work under the Cyberpunk 2077 `r6\scripts` directory.
---

# Cyberpunk 2077 redscript Project Instructions

These instructions apply to all work under the Cyberpunk 2077 `r6\scripts` directory.

## Local Path Aliases

This file uses path aliases so it can be reused on machines with different install locations. When applying these instructions, resolve aliases to the local machine's paths:

- `<GAME_ROOT>` = `C:\Games\steamapps\common\Cyberpunk 2077`
- `<MOD_SCRIPTS_ROOT>` = `<GAME_ROOT>\r6\scripts`
- `<GAME_SOURCES_ROOT>` = `D:\Sources\other\redscript\sources`
- `<REDSCRIPT_CLI>` = `D:\Sources\other\redscript\redscript-cli.exe`
- `<BASE_REDSCRIPTS>` = `<GAME_ROOT>\r6\cache\final.redscripts`
- `<CODEWARE_SCRIPTS_ROOT>` = `<GAME_ROOT>\red4ext\plugins\Codeware\Scripts`
- `<ARCHIVEXL_SCRIPTS_ROOT>` = `<GAME_ROOT>\red4ext\plugins\ArchiveXL\Scripts`
- `<TWEAKXL_SCRIPTS_ROOT>` = `<GAME_ROOT>\red4ext\plugins\TweakXL\Scripts`
- `<MOD_SETTINGS_SCRIPTS_ROOT>` = `<GAME_ROOT>\red4ext\plugins\mod_settings`

If these instructions are copied to another PC, update the values above first. `<MOD_SCRIPTS_ROOT>` is the current Cyberpunk 2077 `r6\scripts` directory that contains installed `.reds` mods and this `AGENTS.md`; `<GAME_SOURCES_ROOT>` is the decompiled/original game script sources directory.

Relative paths such as `RevisedBackpack\RevisedBackpackController.reds` are relative to `<MOD_SCRIPTS_ROOT>` unless another root is stated.

`<REDSCRIPT_CLI>` and `<BASE_REDSCRIPTS>` are required only for compile validation. If either path is missing, skip compile validation and report that it was not run.

`<CODEWARE_SCRIPTS_ROOT>`, `<ARCHIVEXL_SCRIPTS_ROOT>`, `<TWEAKXL_SCRIPTS_ROOT>`, and `<MOD_SETTINGS_SCRIPTS_ROOT>` are optional compile context roots. They are not dependencies of every mod. Keep their conventional paths here, but include them in compile `-s` arguments only when the path exists locally.

## Domain

This project contains Cyberpunk 2077 `.reds` files written in redscript. redscript has Swift-like surface syntax, but it is not Swift. Do not assume Swift rules, standard library APIs, visibility semantics, generics, or type inference behavior unless redscript or the game scripts demonstrate them.

Most scripts here patch or extend the game's existing scripted classes. The primary workflow is:

1. Find the original game class, method, field, enum, or helper in `<GAME_SOURCES_ROOT>`.
2. Match the original signature exactly before wrapping or replacing behavior.
3. Prefer minimal, compatibility-friendly changes in the local `.reds` mod file.
4. Check installed scripts in this directory for existing patches to the same class or method.
5. When a change depends on broader class behavior, read the original class and nearby related scripts before deciding where to patch.

## Reference Sources

Use these local sources before guessing API names or method signatures:

- Game script sources: `<GAME_SOURCES_ROOT>\`
- Codeware script sources: `<CODEWARE_SCRIPTS_ROOT>\`
- ArchiveXL script sources: `<ARCHIVEXL_SCRIPTS_ROOT>\`
- TweakXL script sources: `<TWEAKXL_SCRIPTS_ROOT>\`
- Mod Settings script sources: `<MOD_SETTINGS_SCRIPTS_ROOT>\`
- Existing installed mods: `<MOD_SCRIPTS_ROOT>\`

Use the redscript wiki for language rules and common patterns:

- `https://wiki.redmodding.org/redscript/language/language-features/annotations`
- `https://wiki.redmodding.org/redscript/language/language-features/intrinsics`
- `https://wiki.redmodding.org/redscript/language/language-features/loops`
- `https://wiki.redmodding.org/redscript/language/language-features/strings`
- `https://wiki.redmodding.org/redscript/language/language-features/modules`
- `https://wiki.redmodding.org/redscript/language/language-features/conditional-compilation`
- `https://wiki.redmodding.org/redscript/language/native-types`
- `https://wiki.redmodding.org/redscript/language/built-in-functions/math`
- `https://wiki.redmodding.org/redscript/language/built-in-functions/random`
- `https://wiki.redmodding.org/redscript/language/built-in-functions/utilities`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/safe-downcasting`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/class-constructors`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/hash-maps`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/heterogeneous-array-literals`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/scriptable-systems-singletons`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/delaysystem-and-delaycallback`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/generic-callbacks`
- `https://wiki.redmodding.org/redscript/references-and-examples/common-patterns/persistence`
- `https://wiki.redmodding.org/redscript/references-and-examples/logging`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djkovrik/CP77Mods](https://github.com/djkovrik/CP77Mods) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
