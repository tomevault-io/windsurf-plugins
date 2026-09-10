---
trigger: always_on
description: provides many shared helpers — GUID <-> ObjectState conversion, coordinate/polygon
---

# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repository.

## Project overview

This repository contains the **Tapir Archicad automation package** — an open-source
project (by ENZYME-APD) that extends Graphisoft Archicad with additional JSON
automation commands and tooling. Upstream: <https://github.com/ENZYME-APD/tapir-archicad-automation>.

It consists of two main components:

- **Tapir Archicad Add-On** — in [archicad-addon/](archicad-addon/). A C++ Archicad
  Add-On that registers many new JSON commands on top of the official commands
  provided by Graphisoft. Ready to use; see installation instructions below.
  Full command list: <https://enzyme-apd.github.io/tapir-archicad-automation/archicad-addon>.
- **Tapir Grasshopper Plugin** — in [grasshopper-plugin/](grasshopper-plugin/). A
  Grasshopper (Rhino) plugin that exposes the above commands as visual components so
  non-programmers can use them. Work in progress.

The Add-On uses the **Archicad API (DevKit)**. The API interface headers live under
[archicad-addon/Build/DevKits/](archicad-addon/Build/DevKits/) — e.g.
`archicad-addon/Build/DevKits/AC29/Support` for Archicad 29, with corresponding
folders (`AC25`–`AC28`) for older versions.

The Add-On code is based on Tibor Lorantfy's original
[archicad-additional-json-commands](https://github.com/tlorantfy/archicad-additional-json-commands).

The current version is defined in
[archicad-addon/Sources/AddOnVersion.hpp](archicad-addon/Sources/AddOnVersion.hpp)
and [tools/package_info.json](tools/package_info.json) — keep these in sync.
It is bumped automatically after each release by the monthly release
workflow, so it is not repeated here.

## Repository layout

```
archicad-addon/          C++ Archicad Add-On
  Sources/               Command implementations (*.cpp/*.hpp), one file per command group
  Build/                 CMake build output + DevKits/ (downloaded Archicad API SDKs)
  Tools/                 Build/packaging scripts (CMake helpers, resource compiler, signing)
  Examples/              Python usage examples (one .py per feature) + aclib/ helper
  Test/                  test_examples.py runs Examples against TestProject.pla
  README.md
grasshopper-plugin/      C# Grasshopper plugin (.NET, produces .gha), packaged via Yak
  TapirGrasshopperPlugin/  Components/, Types/, Helps/, Resources/
builtin-scripts/         Bundled automation scripts
branding/                Logos, diagrams
docs/                    Generated docs (archicad-addon command reference)
tools/                   Version bump scripts (update_version.py), package_info.json
  discord-issue-bot/     Discord -> GitHub issue bot (run by .github/workflows/discord_issue_bot.yml)
sandbox/                 Experiments / scratch
.github/workflows/       CI: build checks + release pipelines for both components
```

## Archicad Add-On architecture

- **Entry point:** [archicad-addon/Sources/AddOnMain.cpp](archicad-addon/Sources/AddOnMain.cpp).
  `Initialize()` registers every command, grouped into `CommandGroup`s (Application,
  Project, Element, Attribute, Property, Classification, Navigator, Issue, Revision,
  Design Options, IFC, Library, Teamwork, Favorites, Solid Element Operation, etc.).
  Each `RegisterCommand<T>(group, version, description)` call also records the version
  the command was introduced and a human description used for the generated docs.
- **Commands** are grouped by domain into paired `*.cpp`/`*.hpp` files in `Sources/`
  (e.g. `ElementCommands`, `ElementCreationCommands`, `AttributeCommands`,
  `NavigatorCommands`, `PropertyCommands`, `IFCCommands`). All commands live in the
  `TapirCommand` namespace and are invoked via the Archicad JSON interface.
- **CommandBase** ([Sources/CommandBase.hpp](archicad-addon/Sources/CommandBase.hpp))
  is the base class. It defines the namespace, execution policy, and schema hooks, and
  provides many shared helpers — GUID <-> ObjectState conversion, coordinate/polygon
  helpers, story/floor-index resolution (`GetStories`, `ResolveFloorIndexAndOffset`),
  attribute lookup, and `Create*Response`/`Create*ExecutionResult` builders. Reuse these
  helpers when adding commands; match the existing style.
- **Schemas:** input/response JSON schemas are defined per command
  (`GetInputParametersSchema` / `GetResponseSchema`) plus shared definitions in
  [Sources/SchemaDefinitions.cpp](archicad-addon/Sources/SchemaDefinitions.cpp).
- **UI:** the Add-On adds a menu + palette ([TapirPalette](archicad-addon/Sources/TapirPalette.cpp)),
  an About dialog, and an auto-update/version check ([VersionChecker](archicad-addon/Sources/VersionChecker.cpp)).

### Adding or changing a command

1. Add/modify the command class in the appropriate `Sources/*Commands.{cpp,hpp}`
   (or create a new group file and include it in `AddOnMain.cpp`).
2. Implement input/response schemas; reuse `SchemaDefinitions` and `CommandBase` helpers.
3. Register it in `AddOnMain.cpp` `Initialize()` with the version it is introduced in
   and a clear description.
4. Add a Python example under [archicad-addon/Examples/](archicad-addon/Examples/) and,
   if it should be tested, expected output under `Test/ExpectedOutputs/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ENZYME-APD/tapir-archicad-automation](https://github.com/ENZYME-APD/tapir-archicad-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
