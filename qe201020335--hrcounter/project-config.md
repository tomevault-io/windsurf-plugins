---
trigger: always_on
description: Guidance for AI coding agents working on this repository.
---

# AGENTS.md

Guidance for AI coding agents working on this repository.

## Project

**HRCounter** is a Beat Saber mod (BSIPA plugin) that displays the player's real-time heart rate during gameplay. Author: qe201020335. Source: https://github.com/qe201020335/HRCounter. Current target Beat Saber version is in `HRCounter/manifest.json`.

The mod ingests heart rate from many possible sources, broadcasts the value through a small in-game pipeline, and renders it either via a Counters+ canvas integration or a standalone world-space counter. It also records HR into BeatLeader replays and plays it back on replay viewing.

## Tech stack

- **Language**: C# (see `<LangVersion>` in `HRCounter/HRCounter.csproj`), targets `net472`. The actual runtime is Unity's bundled Mono, not Microsoft .NET Framework — so `HRCounter.csproj` sets `<FrameworkPathOverride>$(BeatSaberDir)\Beat Saber_Data\Managed</FrameworkPathOverride>` to compile against the Mono framework DLLs the game ships with. This avoids runtime surprises from API differences between Mono and the real .NET Framework. Don't disable that override.
- **Plugin framework**: BSIPA + SiraUtil. Versions pinned in `HRCounter/manifest.json` (`dependsOn`).
- **DI**: Zenject (Beat Saber bundles it), wired via SiraUtil installers.
- **UI**: BeatSaberMarkupLanguage (BSML).
- **Optional integrations**: Counters+, BeatLeader, ScoreSaber, YUR.
- **JSON**: Newtonsoft.Json. Custom converters live in `HRCounter/Utils/Converters/`.
- **Asset bundle**: counter prefab is built in the separate `HRCounterBundle/` Unity project and shipped as `HRCounter/Resources/hrcounter`.

## Branching

- `master` — last public release. Often outdated because only stable, non-prerelease code lands here.
- `dev` — **default working branch** for in-progress prerelease work.
- Feature branches are cut from `dev` (e.g. `pulsoid`, `bsml`, `websocket`, `1.42`).
- PRs target `dev`, not `master`.

## Build / run

Standard BSIPA mod build:

1. Restore NuGet packages — SDK references game assemblies via `BeatSaberModdingTools.Tasks` / `BeatSaberDir`.
2. Build the `HRCounter` project. Output goes to the configured Beat Saber install.
3. Launch Beat Saber; logs land in `Logs/_latest.log` under the install root.

The asset bundle (`HRCounter/Resources/hrcounter`) is **not** built as part of the C# project. It is committed to the repo and only rebuilt manually from the `HRCounterBundle/` Unity project when the prefab/font/shaders change. See the `HRCounterBundle/` section below.

## Sub-projects in the repo

These live alongside the main `HRCounter/` plugin project but build/deploy independently.

### `HRCounterBundle/`

Unity project that produces the counter prefab asset bundle. The Unity version is pinned to match Beat Saber's runtime — see `HRCounterBundle/ProjectSettings/ProjectVersion.txt`. Don't upgrade Unity casually.

- Contains `Assets/HRCounter.prefab` (the in-game counter layout — TextMeshPro number + heart icon + replay icon), the custom font (`Heartbit-Bold SDF`), and embedded TextMesh Pro shaders.
- `Assets/Editor/AssetBundleExporter.cs` is the editor script that builds the bundle.
- **Manual build only.** Open the project in Unity, run the exporter, and copy/commit the output to `HRCounter/Resources/hrcounter`. The C# build does not invoke Unity. Only re-export when the prefab layout, font, or shaders change.
- Loaded at runtime by `AssetBundleManager` from the embedded manifest resource.

### `ConfigGenerator/`

React + TypeScript web app (Create React App style, `react-scripts`) for generating the mod's JSON config file outside the game. Uses Fluent UI components. Versions are in `ConfigGenerator/package.json`.

- Entry: `src/index.tsx` → `src/App.tsx` → `src/Components/Main.tsx`.
- Models in `src/models/` mirror the C# config / data source shape so users can build a config without launching the game.
- Helpers: `src/utils/Generator.ts`, `FileSaver.ts`, `EncodingHelper.ts`, `CryptHelper.ts`, `GameSettingsController.ts`.
- Standard CRA scripts: `npm start`, `npm run build`, `npm test`. Not bundled into the plugin — deployed separately as a static site.

### `hrcounter-proxy/`

Cloudflare Worker (TypeScript, Wrangler) that proxies the HypeRate WebSocket for the in-game `HypeRate2` data source.

- Entry: `src/index.ts`. HypeRate logic in `src/hyperate/handler.ts` and `src/hyperate/HypeRate.ts`. Auth in `src/auth.ts` (validates Beat Saber platform user / Steam / Oculus).
- The proxy exists so the mod doesn't ship HypeRate API credentials and so protocol changes can be patched server-side without a mod update. The `HypeRate2` source connects to `wss://hrcounter.skyqe.net/proxy/hyperate`.
- Scripts: `npm run dev` (local Wrangler), `npm run deploy` (push to Cloudflare), `npm test` (Vitest with `@cloudflare/vitest-pool-workers`).
- Config in `wrangler.jsonc`. Type definitions auto-generated via `npm run cf-typegen`.

## Architecture

### Live HR data flow

```
[HR device]
    └─> [DataSource impl: Pulsoid2/HypeRate2/YUR/HTTP/OSC/...]
          └─> IHRDataSource.OnHRDataReceived event
                └─> HRDataManager (marshals to main thread, broadcasts)
                      ├─> LiveHRProvider (caches current HR, fires HRChanged)
                      │     └─> HRCounter (abstract base) → display

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qe201020335/HRCounter](https://github.com/qe201020335/HRCounter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
