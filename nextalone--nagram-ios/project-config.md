---
trigger: always_on
description: You are an experienced, pragmatic software engineering AI agent. Do not over-engineer a solution when a simple one is possible. Keep edits minimal. If you want an exception to ANY rule, you MUST stop and get permission first.
---

You are an experienced, pragmatic software engineering AI agent. Do not over-engineer a solution when a simple one is possible. Keep edits minimal. If you want an exception to ANY rule, you MUST stop and get permission first.

# AGENTS.md

This file guides AI agents working in this repository. It is specific to Nagram-iOS and should be kept in sync with the repo, not with generic Telegram-iOS assumptions.

## Project Overview

Nagram-iOS is a third-party enhancement fork of [Telegram-iOS](https://github.com/TelegramMessenger/Telegram-iOS), targeting Chinese users and aligning selected features with Android [Nagram](https://github.com/NextAlone/Nagram). The goal is to keep the fork easy to rebase onto upstream Telegram while adding Nagram-specific settings, UI, translation, privacy, and interaction features.

Technology stack:

- iOS app code in Swift, Objective-C, Objective-C++, C, and C++.
- Bazel workspace (`WORKSPACE`, `MODULE.bazel`) with a custom Python wrapper at `build-system/Make/Make.py`.
- Xcode/iOS SDK toolchains; expected versions are tracked in `versions.json`, while local build caveats live in `docs/build.md`.
- Telegram modules under `submodules/` (`TelegramCore`, `TelegramUI`, `Display`, `SwiftSignalKit`, `Postbox`, etc.) plus vendored native dependencies under `third-party/`.

## Reference

Important directories:

- `Nagram/` — all new Nagram feature code. Main modules are `Settings/`, `SettingsSignal/`, `SettingsUI/`, `Strings/`, and `Translate/`.
- `Telegram/` — main app target, app extensions, app plist fragments, icons, and app-level Bazel rules.
- `submodules/` — upstream Telegram libraries. Modify only when a feature must integrate with upstream code, and mark the edit.
- `third-party/` — vendored dependencies. Avoid style-only or opportunistic changes here.
- `Tests/` — Bazel test/demo targets; `Tests/AllTests/BUILD` currently includes `//submodules/TgVoipWebrtc:TgCallsTests`.
- `Telegram/Tests/Sources/` — XCUITest sources for the generated Xcode project.
- `docs/` — build notes, UI testing notes, and the Postbox-to-TelegramEngine migration log.

Important files:

- `README.md` and `docs/build.md` — signing modes, build commands, and current local toolchain pitfalls.
- `.bazelrc` — imports gitignored `local.bazelrc`; local signing/provisioning flags belong there.
- `build-system/Make/Make.py` — supported build/test/clean/query entry point.
- `Telegram/BUILD` — app target, plist fragments, Nagram app name, strings, and icon integration.
- `Nagram/Settings/NagramSettings.swift` — central Nagram settings store.
- `Nagram/SettingsSignal/Sources/NagramSettingsSignal.swift` — reactive settings bridge.
- `Nagram/SettingsUI/NagramSettingsController.swift` — Nagram settings UI entry controller.
- `submodules/TelegramUI/Components/PeerInfo/PeerInfoScreen/Sources/PeerInfoSettingsItems.swift` — Settings screen Nagram entry point (`SettingsSection.nagram`, item id `50`).
- `docs/superpowers/postbox-refactor-log.md` — source of truth for the Postbox migration waves.

## Essential Commands

Full app builds are the only reliable validation path for app changes. There is no supported per-module build workflow for this fork.

### Device build hard gate

For any request to build for or install on a physical iPhone, follow this order before selecting a signing mode or starting a build:

1. Read `docs/build.md` sections “签名模式选择”, “真机构建强制预检”, “在 workspace / worktree 构建真机包”, and the matching signing-mode section.
2. Check the connected device, Apple Development identity, main-app and all 6 extension profiles, `build-input/local-configuration.json`, `build-input/codesigning-development/`, Bazel rule/submodule directories, and signing flags in `local.bazelrc`.
3. If the app and all 6 extension profiles are available, use full signing. Never set `disableExtensions` or `disableProvisioningProfiles`.
4. Missing gitignored `build-input` files in an isolated jj workspace do **not** imply free-signing mode. Restore them from the operator-approved private source first; do not inspect another workspace without explicit permission.
5. Use free Apple ID signing only when complete profiles are genuinely unavailable and the user requested that mode. It may disable extensions, but never provisioning profiles.
6. Empty Bazel rule/submodule directories are dependency failures, not signing failures. Do not change signing mode to work around them. If recovery requires a `git` command, request explicit authorization for that exact command under the jj-only policy.
7. Do not claim a device build succeeded until the IPA is produced, installed with `devicectl`, and the install result is verified.

### Build

Simulator-only, codesigning-free setup:

```sh
cat > local.bazelrc <<'EOF'
build --//Telegram:disableProvisioningProfiles
build --//Telegram:disableExtensions
EOF

python3 build-system/Make/Make.py --overrideXcodeVersion \
  --cacheDir ~/telegram-bazel-cache \
  build \
  --configurationPath build-system/appstore-configuration.json \
  --xcodeManagedCodesigning --buildNumber=1 \
  --configuration=debug_sim_arm64 --continueOnError
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextAlone/Nagram-iOS](https://github.com/NextAlone/Nagram-iOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
