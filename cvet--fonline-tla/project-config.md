---
trigger: always_on
description: Project front door for AI maintainers working on **FOnline: The Life After** (TLA). Read this before changing anything. Keep this file as the single source for agent-facing repository practices; `CLAUDE.md` and `.github/copilot-instructions.md` intentionally point here.
---

# FOnline: The Life After - Agent Instructions

Project front door for AI maintainers working on **FOnline: The Life After** (TLA). Read this before changing anything. Keep this file as the single source for agent-facing repository practices; `CLAUDE.md` and `.github/copilot-instructions.md` intentionally point here.

## What This Project Is

- TLA is a multiplayer game built on top of the reusable **fonline-engine** submodule in `Engine/`.
- The split is engine plus game:
  - `Engine/` - upstream engine submodule. Treat as external unless the task explicitly requires an engine change.
  - `Scripts/*.fos` - AngelScript gameplay, dialogs, quests, AI, GUI behavior, and server/client hooks. The folder is flat except for `Scripts/Json/`.
  - `SourceExt/*.cpp` / `SourceExt/*.h` - project-local native C++ extensions registered from `CMakeLists.txt`.
  - `Critters/`, `Items/`, `Maps/`, `Dialogs/`, `Gui/`, `Texts/`, `Resources/` - authored game content and assets.
  - `TLA.fomain` - master engine/game config and `[SubConfig]` profiles.
  - `CMakeLists.txt` / `CMakePresets.json` - build glue. Default local preset is `auto` into `Build/Auto`.
- The user usually converses in Russian; answer the user in Russian unless asked otherwise.
- **Script comment language is Russian** (owner decision 2026-06-20, reversing the prior English-only rule). In `Scripts/*.fos`: code comments and the per-file header block (see [Docs/ScriptStyle.md](Docs/ScriptStyle.md)) are written in Russian, and existing English comments are translated to Russian as files are touched. **Exception:** serialized/contract names stay English — `///@ Property/Enum/Setting/Event/RemoteCall`, proto ids, text-pack keys, and identifiers (renaming them risks save/network/content migration). Agent-facing markdown (`AGENTS.md`, most of `Docs/`) and native C++ (`SourceExt/`) remain English; player-facing text follows the existing localized pack structure.
- Text packs are baked for `russ engl`; `Client.Language = engl` in the default config. When editing player-facing text, preserve the existing pack structure and update both language surfaces when the nearby content expects that.

## Repository Orientation

- `Engine/` - pinned fonline-engine submodule. Do not edit in place for game behavior; advance the SHA in coordinated chunks.
- `Scripts/Content.fos` - generated/baked content declarations. Do not hand-edit.
- `Scripts/GuiScreens.fos` - generated screen bindings. Source of truth: `Gui/*.fogui` plus `Tools/InterfaceEditor/generate_gui_screens.py`. Do not hand-edit unless you also update the owning `.fogui` code as described below.
- `Scripts/GuiScreensExt.fos` - hand-written companion to `GuiScreens.fos`; non-generated GUI logic lives here.
- `Scripts/Sync.fos` - script-side helpers around the engine `Game.Sync(...)` lock primitive for async worker code.
- `Gui/*.fogui` - GUI definitions and embedded screen script code.
- `SourceExt/CommonExtension.cpp` - SHA helpers shared by client/server.
- `SourceExt/ServerExtension.cpp` - server image checks, dialog plumbing, visibility hooks, critter busy/free stubs.
- `SourceExt/ClientExtension.cpp` / `SourceExt/ClientExtension.h` - `Game.FormatTags`, client critter busy/free stubs, `ClientInitHook` + `ClientExtData` (holds the AI control bridge state and embedded-client index).
- `SourceExt/ClientAiBridge.cpp` - localhost TCP line protocol for the AI control bridge (client-side test/automation). Pairs with `Scripts/AiControl.fos` and `Tools/AiControlMcp/`. See [Docs/AiControl.md](Docs/AiControl.md).
- `SourceExt/BakerExtension.cpp`, `SourceExt/DialogBaker.*`, `SourceExt/Dialogs.*` - dialog bake/runtime support.
- `SourceExt/ContentMigration.cpp` - TLA-specific content/data migrations.
- `SourceExt/SHA/` - bundled SHA implementation wrapped as a static library.
- `Tools/Formatter/format_project.py` and `FormatSource.bat` - formatting entry points. VS Code tasks use the Python formatter.
- `Tools/InterfaceEditor/generate_gui_screens.py` - supported GUI screen generator.
- Build/runtime logs are written to repo root: `TLA_Server.log`, `TLA_ServerHeadless.log`, `TLA_Client.log`, `TLA_Baker.log`, `TLA_ASCompiler.log`, `TLA_Mapper.log`, `Build/_bake.log`, `Build/_errors.txt`.
- Built binaries land under `Binaries/`; baked output lives in `Baking/`; cache in `Cache/`; resource trees in `Resources/`.

Adding or removing a native extension file requires updating the relevant `AddEngineSources(...)` block in `CMakeLists.txt` under `COMMON`, `SERVER`, `CLIENT`, or `BAKER`.

## Build And Verify

VS Code tasks in `.vscode/tasks.json` are the authoritative workflow. The same commands can be run directly from the terminal.

Warnings are treated as failures. Keep script compilation, resource baking, native builds, tests, and smoke runs at zero warnings; fix new or existing warnings instead of hand-waving them in the handoff.

| Task | When to use |
| ---- | ----------- |
| `Bake Resources` | After edits in `Scripts/`, `Dialogs/`, `Maps/`, `Items/`, `Critters/`, `Texts/`, `Gui/`, or `TLA.fomain`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cvet/fonline-tla](https://github.com/cvet/fonline-tla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
