---
trigger: always_on
description: Guidance for Claude Code when working on Omacale (`shell/omacale/`).
---

# CLAUDE.md — Omacale

Guidance for Claude Code when working on Omacale (`shell/omacale/`).

## What this is

Omacale is a **Caelestia-style desktop shell for Omarchy**, shipped as one Omarchy shell **bar plugin** (`omacale.bar`). We copy Caelestia's UI and UX; Omarchy is the engine underneath.

- **UI/UX source of truth:** `shell/caelestia_shell/` (a checkout of Caelestia). When something looks or feels different from Caelestia, Caelestia is right and Omacale is the bug.
- **Engine:** Omarchy (`/usr/share/omarchy`, source in `omarchy-repo/`). Data, actions and state come from Omarchy commands and state files.
- **Runtime:** inside the already-running Omarchy shell (Quickshell). **Never start a second Quickshell process.** No C++ build, no changes to the user's Hyprland config beyond the optional `keybinds.lua` and `omacale.lua` snippets.

Other directories under `shell/` (`lacuna-shell`, `ruixen-shell`, `Shibumi-Shell`) are unrelated references. Don't pull from them unless asked.

## The two rules

### 1. Match Caelestia, don't approximate it

Before building or changing any UI, read the Caelestia original and port its structure, not just its look:

| Omacale | Caelestia original (`shell/caelestia_shell/`) |
|---|---|
| `Sidebar.qml` (+ inline `NotifDock`) | `modules/sidebar/Content.qml`, `NotifDock.qml` |
| `NotifGroup.qml`, `NotifItem.qml` | `modules/sidebar/NotifGroup.qml`, `Notif.qml`, `NotifActionList.qml` |
| `NotifPopups.qml` (stack + `ExtraIndicator`), `NotifToast.qml` | `modules/notifications/Content.qml`, `Wrapper.qml`, `Notification.qml`, `components/widgets/ExtraIndicator.qml` |
| `Utilities.qml` (+ inline delete dialog) | `modules/utilities/Content.qml`, `Wrapper.qml`, `RecordingDeleteModal.qml` |
| `QuickToggles.qml` | `modules/utilities/cards/Toggles.qml` |
| `IdleInhibitCard.qml` | `modules/utilities/cards/IdleInhibit.qml` |
| `RecordCard.qml`, `RecordingList.qml` | `modules/utilities/cards/Record.qml`, `RecordingList.qml` |
| `ButtonRow.qml`, `IconButton.qml` | `plugin/src/Caelestia/Components/buttonrow.cpp`, `components/controls/ButtonBase.qml`, `IconButton.qml` |
| `SplitSelect.qml` | `components/controls/SplitButton.qml` |
| `MSwitch.qml`, `MSlider.qml`, `CircularProgress.qml`, `MTextField.qml`, `OutlinedField.qml` | `components/controls/StyledSwitch.qml`, `StyledSlider.qml`, `CircularProgress.qml`, `TextFieldBase.qml`, `StyledTextField.qml` (outlined) |
| `MFlickable.qml`, `MListView.qml`, `FadeFlickable.qml`, `FadeListView.qml`, `MScrollBar.qml` | `components/containers/StyledFlickable.qml`, `StyledListView.qml`, `VerticalFadeFlickable.qml`, `VerticalFadeListView.qml`, `components/controls/StyledScrollBar.qml` |
| `Elevation.qml` | `components/effects/Elevation.qml` |
| `StateLayer.qml`, `Anim.qml`, `CAnim.qml` | `components/StateLayer.qml`, `Anim.qml`, `CAnim.qml` |
| `PopoutContent.qml` (one component per bar popout; `wirelesspassword` and `winfo`, like `traymenu`, are sticky, and hold the keyboard: see `ScreenScope.popoutSticky` / `popoutHeld`) | `modules/bar/popouts/Content.qml`, `Network.qml`, `WirelessPassword.qml`, `kblayout/KbLayout.qml`, `Bluetooth.qml`, `Battery.qml`, `AudioPopout.qml`, `LockStatus.qml`, `TrayMenu.qml`, `ActiveWindow.qml` |
| `Workspaces.qml`, `SpecialWorkspaces.qml`, `ActiveIndicator.qml` | `modules/bar/components/workspaces/Workspaces.qml`, `SpecialWorkspaces.qml`, `ActiveIndicator.qml` |
| `WindowInfo.qml`, `WinfoPreview.qml`, `WinfoDetails.qml`, `WinfoButtons.qml` (the held `winfo` popout, detached from `activewindow`; also IPC `windowInfo`) | `modules/windowinfo/WindowInfo.qml`, `Preview.qml`, `Details.qml`, `Buttons.qml`, and `bar/popouts/Wrapper.qml` `detach("winfo")`. One deliberate difference: Kill closes the panel, since it follows the active window and would otherwise turn to the next one |
| `KbService.qml` | `modules/bar/popouts/kblayout/KbLayoutModel.qml`, the layout half of `services/Hypr.qml` |
| `Tk.qml` | Caelestia `Tokens` (`plugin/src/Caelestia/Config/tokens.hpp`, `appearanceconfig.hpp`) |
| `Colours.qml` | Caelestia `Colours` (M3 palette from the Omarchy theme accent; or, with Settings › Style › Palette › Omarchy, the theme's own colours on the M3 roles) |
| `WallLuminance.qml` | Caelestia's `ImageAnalyser` (`plugin/src/Caelestia/Images/imageanalyser.cpp`), feeding `Colours.wallLuminance` |
| `ScreenScope.qml` + `shaders/blob.frag` | `modules/drawers/` (`Panels.qml`, `Backgrounds`) and its `blob.frag` |
| `Dashboard.qml`, `Launcher.qml`, `Session.qml`, `Settings.qml` (Nexus) | `modules/dashboard`, `launcher`, `session`, `nexus` |
| `LockUi.qml`, `LockContent.qml` | `modules/lock/LockSurface.qml`, `Content.qml` |
| `LockCenter.qml`, `LockPassword.qml`, `LockMessage.qml` | `modules/lock/Center.qml`, `center/Clock.qml`, `ProfilePic.qml`, `PasswordInput.qml`, `InputField.qml`, `StateMessage.qml` |
| `LockWeather.qml`, `LockFetch.qml`, `LockMedia.qml`, `LockResources.qml`, `LockNotifs.qml` | `modules/lock/WeatherInfo.qml` (+ `weather/`), `Fetch.qml`, `Media.qml`, `Resources.qml`, `NotifDock.qml` |
| `NetworkPage.qml`, `NetworkDetail.qml` | `modules/nexus/pages/NetworkPage.qml`, `common/NetworkList.qml`, `network/NetworkDetailPage.qml` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AyushKr2003/omacale](https://github.com/AyushKr2003/omacale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
