---
trigger: always_on
description: ValidateSuite.swift
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Moblin is a free iOS/iPadOS IRL streaming app (Swift/SwiftUI) targeting Twitch, YouTube, Kick, Facebook, and OBS Studio. Includes an Apple Watch companion app, Live Activity extension, home screen widget, screen recording extension, and a SolidJS web remote control frontend.

## Building

1. Copy `User.template.xcconfig` → `Config/User.xcconfig`, set `DEVELOPMENT_TEAM` and `BASE_PRODUCT_BUNDLE_IDENTIFIER`.
2. `open Moblin.xcodeproj` — wait for SPM packages to resolve.
3. `Command + B` to build; `Command + R` to run on device.

Tests run via Xcode (`Command + U`). Test files live in `MoblinTests/` named `*Suite.swift`.

## Make targets

```sh
make style           # swiftformat + oxfmt (auto-fix)
make style-check     # lint-only, no writes
make lint            # swiftlint + oxlint + xcstringslint
make spell-check     # codespell
make periphery       # dead code detection

make web-remote-control-frontend-prepare   # npm install
make web-remote-control-frontend-build     # tsc check + vite build → embeds into Moblin/RemoteControl/Web/
```

Required tools: `swiftlint swiftformat periphery codespell oxfmt oxlint` (Homebrew).

## Key conventions

- `swiftformat` line width: 110 chars, Swift 5.9 mode.
- `swiftlint --strict`. Many rules disabled — see `.swiftlint.yml`. `force_cast` and `force_try` are disabled.
- Tesla Protobuf files (`Moblin/Integrations/Tesla/Protobuf/`) excluded from formatting and periphery.
- All localizations in `Common/Localizable.xcstrings` (not `.strings` files). Lint target: `xcstringslint`.
- `moblin://` URL scheme used for settings import — spec in README.

---

## Full codebase map

### Top-level targets

| Directory | Target |
|-----------|--------|
| `Moblin/` | Main iOS/iPadOS app |
| `Moblin Watch/` | watchOS companion |
| `Moblin Widget/` | Home screen widget |
| `Moblin Live Activity/` | Live Activity extension |
| `Moblin Screen Recording/` | Screen recording broadcast extension |
| `Common/` | Shared Swift + SwiftUI across all targets |
| `MoblinTests/` | Unit/integration tests |
| `WebRemoteControlFrontend/` | SolidJS web remote control |
| `Config/` | Xcode build configs (`*.xcconfig`) |
| `utils/` | Python utility scripts (translations, xliff, xcstringslint) |
| `docs/` | Documentation and screenshots |

---

### `Common/`

```
Localizable.xcstrings                  # All app strings / translations
Various/
  AudioLevel.swift
  AVAudioPCMBuffer+Extension.swift
  CMBlockBuffer+Extension.swift
  CMFormatDescription+Extension.swift
  CMSampleBuffer+Extension.swift
  CommonUtils.swift
  Validate.swift
View/
  StreamOverlayIconAndTextView.swift
  StreamOverlayTextView.swift
  ThermalStateView.swift
```

---

### `Moblin/` — Main app

#### Entry point
```
MoblinApp.swift                        # @main SwiftUI app entry
```

#### `Moblin/Various/Model/` — Central state (all @Observable extensions on Model)
```
Model.swift                            # Root @Observable class
ModelAppIntents.swift
ModelAppleWatch.swift
ModelAudio.swift
ModelAutoSceneSwitcher.swift
ModelBlackSharkCoolerDevice.swift
ModelBluetooth.swift
ModelCamera.swift
ModelCatPrinters.swift
ModelChat.swift
ModelChatBot.swift
ModelDisconnectProtection.swift
ModelDjiDevice.swift
ModelFaceBackgroundImage.swift
ModelGameController.swift
ModelGimbal.swift
ModelKeyboard.swift
ModelKick.swift
ModelLiveActivity.swift
ModelLocation.swift
ModelMacros.swift
ModelMediaPlayer.swift
ModelMoblink.swift
ModelNavigation.swift
ModelObs.swift
ModelPictureInPicture.swift
ModelRecording.swift
ModelRemoteControl.swift
ModelReplay.swift
ModelRistServer.swift
ModelRtmpServer.swift
ModelRtspClient.swift
ModelScene.swift
ModelScoreboard.swift
ModelScreenCapture.swift
ModelSettingsImportExport.swift
ModelSettingsUrl.swift
ModelSnapshot.swift
ModelSoop.swift
ModelSpeechToText.swift
ModelSrtlaServer.swift
ModelStealthMode.swift
ModelStore.swift
ModelStream.swift
ModelStreamWizard.swift
ModelTesla.swift
ModelTextToSpeech.swift
ModelTwitch.swift
ModelVideoPreview.swift
ModelWebBrowser.swift
ModelWhepClient.swift
ModelWhipServer.swift
ModelWiFiAware.swift
ModelWorkout.swift
ModelWorkoutDevice.swift
ModelYouTube.swift
ModelZoom.swift
Chat/
  ChatProvider.swift
```

#### `Moblin/Various/Settings/` — JSON-serializable persistent settings (separate from runtime state)
```
Settings.swift                         # Root settings object
SettingsAudio.swift
SettingsCatPrinter.swift
SettingsChat.swift
SettingsDebug.swift
SettingsDeepLinkCreator.swift
SettingsDjiDevice.swift
SettingsGameController.swift
SettingsGimbal.swift
SettingsGoPro.swift
SettingsIngests.swift
SettingsKeyboard.swift
SettingsLocation.swift
SettingsMacros.swift
SettingsMoblink.swift
SettingsNavigation.swift
SettingsQuickButtons.swift
SettingsRemoteControl.swift
SettingsScene.swift
SettingsSelfieStick.swift
SettingsStream.swift
SettingsTalkback.swift
```

#### `Moblin/Various/Storages/` — File-backed asset storage
```
AlertMediaStorage.swift
FileStorage.swift
ImageStorage.swift
LogsStorage.swift
MediaPlayerStorage.swift
PngTuberStorage.swift
RecordingsStorage.swift
ReplaysStorage.swift
ReplayTransitionsStorage.swift
StreamingHistory.swift
VTuberStorage.swift
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eerimoq/moblin](https://github.com/eerimoq/moblin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
