---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Teak Unity SDK — a cross-platform Unity plugin providing push notifications, deep linking, rewards, and marketing channel management. Ships as a `.unitypackage` and a UPM package. The C# runtime code wraps native SDKs for iOS (`libTeak.a`), Android (`teak.aar`), and WebGL (`Teak.jslib`).

## Build Commands

The build system uses **Ruby Rake** (Ruby 3.1.6 via rbenv).

```bash
# Install Ruby dependencies
bundle install

# Full build (default): downloads native SDKs, generates version files, creates Teak.unitypackage
bundle exec rake

# Individual build targets
bundle exec rake build:android   # Download Android AAR + generate Version.java
bundle exec rake build:ios       # Download iOS framework + generate teak_version.m
bundle exec rake build:package   # Generate TeakVersion.cs + create Teak.unitypackage

# Build UPM package (output in temp-upm-build/)
bundle exec rake upm:build

# Build against local native SDK checkouts (expects ../teak-android and ../teak-ios)
BUILD_LOCAL=true bundle exec rake
```

**Code formatting:**
```bash
bundle exec rake format
# Runs: astyle --project --recursive Assets/*.cs --exclude=Assets/Teak/Editor/iOS/Xcode
```

**Documentation generation** (requires Node.js, yarn, doxygen):
```bash
yarn docs
```

**Running tests:**
```bash
# Via Unity CLI (project version: see ProjectSettings/ProjectVersion.txt)
/Applications/Unity/Hub/Editor/<VERSION>/Unity.app/Contents/MacOS/Unity \
  -executeMethod TeakTestRunner.RunAll \
  -logFile unity.test.log -quit -nographics -batchmode \
  -projectPath .
```

Tests use a custom runner (`Assets/Tests/Editor/TeakTestRunner.cs`) with `[TeakTestFixture]` / `[TeakTest]` attributes and `TeakAssert` helpers. Test files live in `Assets/Tests/Editor/`. Since there are no `.asmdef` files, all Editor scripts (including build post-processors) compile into `Assembly-CSharp-Editor`, so tests can access `internal` members directly.

## Architecture

### Platform Abstraction via Preprocessor Directives

All platform-specific code uses `#if UNITY_EDITOR / UNITY_ANDROID / UNITY_IPHONE / UNITY_WEBGL` blocks. The `Teak` class and related types contain parallel implementations for each platform within the same files. Editor builds stub out native calls.

### Key Files

| File | Purpose |
|------|---------|
| `Assets/Teak/Teak.cs` | Main singleton `MonoBehaviour` — user identification, event tracking, rewards |
| `Assets/Teak/Teak.Channel.cs` | Notification channel opt-in/out, categories |
| `Assets/Teak/Teak.Notification.cs` | Schedule/cancel notifications |
| `Assets/Teak/Teak.Operation.cs` | Async operation wrapper around native SDK futures |
| `Assets/Teak/Teak.UserData.cs` | User data model (push/email/SMS status) |
| `Assets/Teak/TeakNotification.cs` | Notification metadata from native layer |
| `Assets/Teak/TeakReward.cs` | Reward claim model and status enum |
| `Assets/Teak/TeakSettings.cs` | `ScriptableObject` for editor config (App ID, API Key) |
| `Assets/Teak/TeakVersion.cs` | Auto-generated from `Templates/TeakVersion.cs.template` — do not edit |
| `Assets/Teak/MiniJSON.cs` | Bundled JSON serializer (namespace `MiniJSON.Teak`) |
| `Assets/Teak/Plugins/iOS/` | iOS native binary, init code, privacy manifest |
| `Assets/Teak/Plugins/Android/` | Android AAR, version class, androidlib wrapper |
| `Assets/Teak/Plugins/WebGL/Teak.jslib` | JavaScript interop loading `teak.min.js` from CDN |
| `Assets/Teak/Editor/` | Build post-processors for Xcode and Android manifest |

### Patterns

- **Singleton access:** `Teak.Instance` lazily creates a `TeakGameObject` with `DontSave` flag.
- **Partial classes:** `Teak` is split across `Teak.cs`, `Teak.Channel.cs`, `Teak.Notification.cs`, `Teak.Operation.cs`, `Teak.UserData.cs`.
- **Async operations:** `Teak.Operation` wraps platform-specific futures and fires `OnDone` callbacks with JSON results.
- **Version compat defines:** `TeakPreProcessDefiner.cs` adds `TEAK_2_0_OR_NEWER` through `TEAK_4_3_OR_NEWER` scripting defines at build time.

### Version Management

- `VERSION` file at repo root contains the Unity SDK version (e.g. `4.3.10`). Updated at release time, not during development.
- `native.config.yml` specifies native SDK versions for iOS and Android.
- Template files in `Templates/` are rendered by Mustache during `rake` build to generate `TeakVersion.cs`, `Version.java`, and `teak_version.m`. Do not edit these generated files directly.
- **Tagging**: The `teak/tag-promote` CI orb parses the HEAD commit message for `Promote to: X.Y.Z` and creates + pushes a git tag.

### Release Flow

```
# 1. Update VERSION file to new version (e.g. 4.3.10)
# 2. Move docs/modules/changelog/unreleased.yaml → versions/X.Y.Z.yaml
# 3. Create fresh unreleased.yaml
# 4. Inline relevant native SDK changes as regular bug/enhancement/new entries
git commit -m "Promote to: X.Y.Z.rc0"
git push
# CI: orb detects commit message → tags X.Y.Z.rc0 → tagged-build workflow → deploy to S3
```

RC → final release flow:
- **rc0**: Promotes with current native SDK versions (may be betas). Used for integration testing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GoCarrot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
