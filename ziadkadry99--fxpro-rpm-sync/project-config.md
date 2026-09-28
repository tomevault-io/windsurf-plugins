---
trigger: always_on
description: (Called "Simagic RPM Sync" before v0.1. On first start it copies the old settings file to the new name.)
---

# FXPro RPM Sync — SimHub plugin

(Called "Simagic RPM Sync" before v0.1. On first start it copies the old settings file to the new name.)

Keeps a Simagic wheel's rev lights (built and tested on an **FX Pro** on an Alpha EVO base) matched to the car being
driven, in every game SimHub supports. SimHub supplies the car/RPM data; the plugin pushes per-car LED settings into
**SimPro Manager 3** through its undocumented local API, and SimPro sends them to the wheel.

The FX Pro has **no native SimHub support** (SimPro 3.2.1's "SimHub via base CANFD" is Zeus wheels only), which is
why this goes through SimPro.

## Build / deploy

```
dotnet build -c Release            # builds net48 DLL and copies it into C:\Program Files (x86)\SimHub\
dotnet build -c Release -p:DeployToSimHub=false   # build only
```

- **SimHub must be closed** to deploy (it locks the DLL). The copy step uses `ContinueOnError`, so check with
  `cmp "C:/Program Files (x86)/SimHub/User.FXProRpmSync.dll" bin/Release/net48/User.FXProRpmSync.dll`.
- References SimHub's own DLLs (`Private=False`); `SIMHUB_INSTALL_PATH` overrides the default path.
- SimHub asks to enable the plugin on first start. Log lines are prefixed `[FXProRpmSync]` in `SimHub\Logs\SimHub.txt`.
- Settings persist in `SimHub\PluginsData\Common\FXProRpmSyncPlugin.GeneralSettings.json` (SimHub rewrites it
  on exit, so edit it only while SimHub is closed). Car data cache: `SimHub\PluginsData\Common\FXProRpmSync\`.

## Files

| File | What |
|---|---|
| `FXProRpmSyncPlugin.cs` | SimHub plugin + settings. DataUpdate detects car changes; a background worker does all HTTP. Overrides API, SimHub actions/properties, preset capture/restore. |
| `SimProClient.cs` | SimPro local API client. |
| `DashSwitcher.cs`, `DashCatalog.cs`, `DashSection.cs` | Dash per car: switching/learning logic, SimPro's dash names + preview pictures, settings section. |
| `SimGameFeed.cs`, `SimProTelemetry.cs`, `SimHubFeedMapper.cs`, `FeedSection.cs`, `SimGameStub/` | Dash values from SimHub: SimPro "SimGame" shared memory + simgame.exe stub, struct layout, SimHub -> struct mapping, settings section. |
| `CarLedDatabase.cs` | Lovely Car Data download/cache, exact + series/team matching. |
| `RpmLayout.cs` | `RpmLayout` (15 LEDs in real RPM + flash + optional per-gear), `CarOverride`. |
| `RpmLightsMapper.cs` | RpmLayout ⇄ SimPro `rpm_lights` JSON, percent rounding, palette snapping, fingerprints. |
| `LedPatterns.cs` | Built-in fallback patterns, color schemes, palette. |
| `SettingsControl.cs`, `OverridesSection.cs`, `LedStrip.cs` | Settings UI (WPF built in code, no XAML) with animated LED previews. |
| `Resources/menu-icon.png` | SimHub menu icon (embedded resource). Traced from Simagic's product photo; `assets/fxpro-outline.svg` is the vector. |

## Releases

GitHub: https://github.com/ziadkadry99/FXPro-RPM-Sync (GPL-3.0). CI can't build it because SimHub's DLLs aren't
redistributable, so releases are built locally: bump `<Version>` in the csproj, `dotnet build -c Release
-p:DeployToSimHub=false`, zip `User.FXProRpmSync.dll` + `README.md` + `LICENSE` as `FXProRpmSync-vX.Y.zip`, and upload
it with `gh release create vX.Y`.

## Pipeline (per car change)

1. `DataUpdate` builds a `Target` (game, CarId, SimHub max RPM / redline); worker picks it up.
2. Read the selected SimPro preset's `rpm_lights`; get/capture the **original** (see "Preset capture").
3. Scale max = **SimPro's game max RPM** (`game_get_running_list` → `maxCarSpeed`), fallback SimHub's.
4. Base layout (real RPM):
   - car in Lovely Car Data → `RpmLayout.FromProfile` (real pattern/colors/shift point; per-gear only if the
     wheel supports it);
   - else → the chosen fallback style (`RpmLightsMapper.FromStyle`): generated pattern or the preset's own pattern,
     shift point = SimHub's redline (a guess: 95% of max unless set in SimHub Car Settings), no flash by default.
5. Per-car override (`CarOverride.Apply`): Offset (shift everything ±rpm), Custom (15 LEDs + flash set by hand),
   or Pattern (a fallback pattern for this car, keeping the car's own shift point).
6. `RpmLightsMapper.ToSimPro` → `preset_set_dev_config` (applies live; not saved into the preset).
7. On SimHub exit (or "Restore"), the original preset lights are pushed back.

## SimPro Manager 3 local API (reverse engineered, SimPro V3.2.2)

- `POST http://127.0.0.1:4010/simpro/api/v3/<method>`, JSON body, **no auth**. Response `{status, message, result}`.
- The UI is a CEF web app served from the same server; method names are an enum in `/simpro/js/main.*.js`
  (`const ye={GET_DEVICE_LIST:"get_device_list",...}`), called via `je(ye.X, params)`. Grep that bundle to find
  how the UI calls anything (use Python, not grep: it's one 2 MB line).
- Useful methods: `get_device_list`, `preset_get_selected_dev_config` / `preset_get_dev_config_list` /
  `preset_get_dev_config` / `preset_set_dev_config` / `preset_save_dev_config`, `game_get_running_list`,
  `get/set_game_auto_switch_preset`, `dash_get_list` / `dash_add` / `dash_modify` / `dash_apply` / `dash_preview` /
  `get_dev_dash_page` / `get_dash_category` / `get_dash_device_size`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ziadkadry99/FXPro-RPM-Sync](https://github.com/ziadkadry99/FXPro-RPM-Sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
