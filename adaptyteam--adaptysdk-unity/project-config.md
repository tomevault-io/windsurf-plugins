---
trigger: always_on
description: This file provides guidance to coding agents working in this repository. It is the single copy —
---

# AGENTS.md

This file provides guidance to coding agents working in this repository. It is the single copy —
`CLAUDE.md` points here rather than repeating any of it, so nothing has to be kept in sync.

## Project Overview

Adapty Unity SDK — a C# wrapper around native [Adapty iOS SDK](https://github.com/adaptyteam/AdaptySDK-iOS) (Swift/SPM) and [Adapty Android SDK](https://github.com/adaptyteam/AdaptySDK-Android) (Kotlin/Maven). Provides in-app purchase management, flow (paywall) rendering, onboarding flows, and subscription analytics for Unity apps. Current SDK version is defined in `Packages/com.adapty.unity-sdk/Runtime/Adapty.cs` (`Adapty.SDKVersion`).

## Build & Development

This is a **Unity project** (Unity 6000.x) — the player is built and tested through the Unity Editor. The JSON layer is the exception: `tests/` links the SDK sources into a plain .NET project, so it needs neither the Editor nor a licence.

The package declares **Unity 2022.3 and newer** as `unity` in `package.json`, and that floor is what Editor-facing code may assume: `AdaptyDependencies` uses `Client.AddAndRemove` and `PackageInfo.FindForAssembly`, neither of which exists all the way back (`AddAndRemove` arrived after 2020.3).

The install path is verified on the floor — `.unitypackage` import into a clean project, then `Adapty SDK > Install Dependencies`, then a compile, all on 2022.3.62f3. Everything else runs on Unity 6. Keep the changelog and `MIGRATION-v3.17-to-v4.1.md` wording matching that split; do not widen it to claim device or build coverage on 2022.3. One trap when re-verifying: recent 2022.3 builds are Extended LTS and refuse to launch without an Industry or Enterprise licence, so pick a build below that cutoff (62f3 works).

**Run the JSON layer tests:**
```bash
dotnet test tests/AdaptySDK.NextTests/AdaptySDK.NextTests.csproj
```
The layer branches on `UNITY_IOS` / `UNITY_ANDROID` and each platform has its own approved snapshots, so a change to it has to pass all four legs: add `-p:AdaptyPlatform=UNITY_IOS` or `-p:AdaptyPlatform=UNITY_ANDROID` for two of the others. Asking for no platform means `UNITY_EDITOR`, which the projects default to — the Editor is a configuration of its own and has to say so, since it is what selects the no-op bridge; an empty define set is a state Unity never produces. The fourth is `-p:AdaptyPlatform="UNITY_IOS%3BADAPTY_KIDS_MODE"` — `%3B` escapes the `;` inside the MSBuild property value. Kids Mode is the only shipped define that changes the wire format rather than which sources compile, and its whole visible effect here is the forced `apple_idfa_collection_disabled`: three configuration requests have a second approved form under `-kids`, and `RequestParityTests.Configured` is what picks it. Nothing else in the layer moves with the define — if a fourth snapshot ever needs a `-kids` form, that is a change in blast radius worth understanding before approving it. `ADAPTY_UPDATE_SNAPSHOTS=1` rewrites the approved files instead of failing. CI runs the same matrix in `.github/workflows/json-layer-tests.yml`.

**Build .unitypackage for distribution:**
```bash
cd deploy && ./build_unitypackage.sh       # export into deploy/output/
cd deploy && ./build_unitypackage.sh -p    # export, then move it to the repository root
```
The export runs a second Editor in batch mode over a throwaway staging project: `Runtime/` copied to `Assets/AdaptySDK`, with `Editor/` merged into the `Editor` folder `Runtime` already contributes for `AdaptySDKDependencies.xml`. `.meta` files come across, so asset GUIDs survive and an upgrade lands on the files it replaces rather than beside them. The staging manifest is written from the package's own `dependencies`, so the export cannot be compiled against a Newtonsoft the package does not declare. `deploy/output/` is ignored by git; `Releases/` is tracked, and is where the release flow puts each artifact it publishes. It is not a complete history — `4.0.0-beta.1` was tagged without one — so it is not the source of the exact bytes of every past release.

**Publish a release:** `deploy/release_unitypackage.sh` builds, moves the artifact into `Releases/`, commits, tags, pushes, and creates the GitHub release. Read its header before running it — it names three things it deliberately does not check, and each of them can publish something you did not mean to. `--dry-run` prints every command it would run and touches nothing.

**Android wrapper (Java):** Built separately via Gradle in `adaptyandroidwrapper/`:
```bash
cd adaptyandroidwrapper && ./gradlew :unitywrapper:build
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adaptyteam/AdaptySDK-Unity](https://github.com/adaptyteam/AdaptySDK-Unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
