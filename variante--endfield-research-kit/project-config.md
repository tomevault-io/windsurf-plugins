---
trigger: always_on
description: Agent notes for this repo. User-facing usage belongs in `README.md`.
---

Agent notes for this repo. User-facing usage belongs in `README.md`.

## Active Scope

Keep root-level docs and workflow guidance focused on:

- the static WebUI in `webui/`

Fold durable observations, conclusions, and recovery status into the existing
topic documents under `memory/`. Do not recreate one-file-per-investigation or
dated status snapshots. Generated inventories belong in `reports/`, and
disposable evidence belongs in `scratch/` or `tmp/`.

## Commands

```bat
.\setup_first_time.bat
.\export.bat
.\export.bat --with-assets
.\export.bat --export-from-game
.\build_updates.bat
.\build_updates.bat --init-build
.\build_updates_by_patch.bat --init-baseline
.\build_updates_by_patch.bat --check
.\build_updates_by_patch.bat
.\export_assets.bat
python serve.py
python serve.py 9000
```

Before starting a WebUI server, check whether the default
`http://127.0.0.1:8765/` server is already running. Reuse the existing default
server instead of starting another `serve.py` process on `8765` or a custom
port, unless the user explicitly asks for a second server.

Root wrapper scripts load `endfield_paths.bat` before parsing arguments. That
file sets the repeated local defaults for `ENDFIELD_GAME_ROOT`,
`ENDFIELD_PREVIOUS_EXPORT_ROOT`, and `ENDFIELD_EXPORT_ROOT`; explicit path
flags still override it for one-off commands.

`setup_first_time.bat` is the user-facing all-in-one first-time setup path. It
initializes `tools/AnimeStudio`, builds the AnimeStudio CLI, verifies the
integrated AnimeStudio VFS/audio commands, runs `export.bat --export-from-game`,
prints optional `export_assets.bat --export-from-game` and
`build_updates.bat --init-build` follow-up commands, then starts or reuses the
default WebUI server. Pass `--no-serve` when setup should finish without
starting `serve.py`.

`export.bat` is the canonical Story/Text Tables and curated semantic-view WebUI rebuild from an existing
`export_full/`. It verifies that `export_full/` matches the current installed
`Endfield_Data` fingerprints before the long WebUI builders run, then builds CN
Story/Text Tables data by default. It does not export from installed game data by
default. Pass `--export-from-game` only when the user explicitly asks to refresh
`export_full/` and run the story export tools. Pass `--with-assets` to also
rebuild asset indexes and relink/decode CN audio after generated conversations
are rebuilt. Combining `--export-from-game --with-assets` runs one AnimeStudio
Story+asset export instead of separate Story and asset exporter invocations.
After semantic-view builders and any requested asset/audio work, `export.bat`
rebuilds `reports/source_graph/endfield_source_graph.sqlite` and then builds the
Presentation and Combat views. Presentation emits a stable empty degraded
payload when its graph is missing or older than its generated inputs. The Combat builder refuses graph edges when the database predates
its Gameplay/manifest/asset/AbilityEntity/CharacterTemplate inputs and records a
visible degraded-mode reason instead of treating stale edges as direct.
`export.bat` does not refresh `webui/overrides/story_order.json`; active Story
order is user-managed there, while OCR recovery writes proposed order references
under `webui/data/story_order_ocr.json`. Every `export.bat` run writes a
wall-time and process-tree RAM benchmark under `reports/export/benchmarks/` and updates
`reports/export/export_benchmark_latest.md/json`.
Use `build_updates.bat` for the standalone Updates feed comparison. Use
`build_updates.bat --init-build` for first-time/baseline-only builds where the
Updates feed should be baselined instead of reporting changes. It reads the
previous/current export roots from `endfield_paths.bat` by default, tracks
WebUI-facing exported text JSON plus exported image/model/video assets and
decoded audio, and accepts explicit root flags for one-off comparisons. Pass
`--skip-audio-updates` to omit decoded audio while keeping other asset entries.
Pass `--skip-asset-updates` only for a text-only update feed.
Use `export_assets.bat` for WebUI Assets tab indexes, compact Story media
lookup, and CN audio relinking from existing decoded assets when Story is
already current. Pass `--export-from-game` only when the user explicitly asks to
run the default full AnimeStudio image/model decode, `Material` JSON, and CN
audio decode from installed game data first. Prefer
`export.bat --export-from-game --with-assets` when Story and assets both need an
installed-game refresh. Pass `--webui-assets` for the lean WebUI-referenced
Texture2D mode, or `--debug-assets` for exhaustive AnimeStudio diagnostics.
Use direct `scripts/build_audio.py` runs for non-CN languages or audio-only
maintenance. The audio builder writes shared SFX/music once under
`export_full/structured/Audio/shared/` and language voice under
`export_full/structured/Audio/<LANG>/`, parses Wwise bank event-to-media links, and post-processes generated
conversation JSON with playable `audioSrc` links. The default exporter mode is
`--animestudio-type-job-mode auto`: it merges map-filtered JSON, runs broad Story
JSON types sequentially in isolated processes, and keeps map-filtered asset
conversion sharded; use `parallel` only when comparing concurrent per-type jobs.

Useful direct commands:

```bat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Variante/endfield_research_kit](https://github.com/Variante/endfield_research_kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
