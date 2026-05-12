---
trigger: always_on
description: handles downloads; HTTP fallback lives in `downloadTtsWithCompanions` and
---

# Contributor & agent guide

This file is the contract for both human contributors and AI coding agents
working on the JarvisQ codebase. Read it fully before touching files.

## Project in one paragraph

JarvisQ is an Expo + React Native consumer of the Tether **`@qvac/sdk`**
that ships an on-device voice assistant (STT → LLM → TTS). The codebase
follows hexagonal / ports-and-adapters architecture so the same core can power
mobile today and desktop (Windows / macOS / Linux) tomorrow. See `README.md`
for the full architecture map.

## Non-negotiable rules

1. **Do not fork or wrap `@qvac/sdk`.** It is consumed directly from
   `node_modules`. Upgrading must remain a `package.json` bump — never add a
   compatibility shim that requires maintenance on SDK updates. The one
   sanctioned exception is `patches/@qvac+sdk+<version>.patch`, applied
   automatically via `patch-package` in `postinstall`; when bumping the SDK,
   refresh the patch (or drop it if upstream fixed the issue) rather than
   introducing wrapper code.
2. **`src/core/` is platform-free.** No `expo-*`, `react-native-*`,
   `@react-native-*`, Node-only, or Bare-only imports are permitted in
   `src/core/`. New capabilities enter the core only through a port under
   `src/core/ports/`; the concrete implementation lives in
   `src/platform/<target>/`.
3. **No hardcoded paths or constants at call sites.** Paths derive from
   `IFileSystem.documentDirectory` and `AppConfig`. Model URLs live in
   `src/core/config/HttpModelSources.ts`; model selection lives in
   `src/core/config/ModelConfig.ts`.
4. **English code and comments only.** Identifiers, comments, commit
   messages, PR descriptions — all English. User-facing UI copy is the only
   exception.
5. **Android works and must keep working.** Verify with `npm run typecheck`
   at minimum. Do not remove the Expo mobile adapters or the
   `bootstrapMobile()` wire-up.
6. **Additive refactors.** When introducing a port, keep the existing public
   call surface stable where practical so feature work and infra work can be
   reviewed independently.

## Where to add things

| Change | Location |
|--------|----------|
| New on-device capability (e.g. embeddings) | `src/core/inference/` + a port if it has platform-specific parts |
| New persisted data | `src/data/repositories/` (uses `IDatabase`) |
| New user setting | `src/domain/SettingsStore.ts` + `src/domain/types.ts` |
| New screen | `src/app/` (Expo Router file) |
| New shared UI primitive | `src/ui/components/` |
| New platform target | new folder under `src/platform/<target>/` with adapters + `bootstrap.ts` |
| New HTTP model URL | `src/core/config/HttpModelSources.ts` (pin a commit SHA) |

## Ports cheat-sheet

The following live under `src/core/ports/`. Keep their surface minimal — only
add methods when a real consumer needs them.

- `IAudioRecorder` — microphone + VAD.
- `IAudioPlayer` — PCM output.
- `IFileSystem` — `getInfo`, `makeDirectory`, `move`, `download`,
  `documentDirectory`.
- `IKeyValueStore` — `getItem` / `setItem` / `removeItem`.
- `IDatabase` — `exec`, `run`, `getFirst`, `getAll`.
- `IHaptics` — short tactile feedback for UI events.
- `IPermissions` — runtime permission checks/requests (microphone today).
- `INetworkInfo` — reachability probe used by `loadWithFallback` to decide
  between P2P registry and HTTPS fallback. A fetch-based default lives in
  `src/core/net/FetchNetworkInfo.ts` and is reused by both mobile and desktop.

### TTS engine contract (`ITtsService`)

Unlike the other ports, the TTS service is not pinned to one implementation
at startup — `VoicePipeline` receives the concrete `ITtsService` via
`PipelineDeps` and the app chooses between `TtsService` (Supertonic) and
`SystemTtsService` (`expo-speech`) based on `AppSettings.ttsEngine`. The
interface is intentionally play-oriented (not synthesize-then-play):

- `speak(text, audioPlayer, options?)` — plays text and resolves on
  completion. PCM engines (Supertonic) write into `audioPlayer`; native
  engines (System) ignore it and manage their own playback.
- `stop()` — aborts current playback without unloading.

Do not leak `synthesize() → Float32Array` back onto this port: native
engines cannot return PCM, and adding it would force branching in
`VoicePipeline.drainTtsQueue`.

Resolve adapters at runtime through `getPlatform()` from
`@core/platform/PlatformContainer` — never `new ExpoX()` from a core file.

## Verification before committing

```bash
npm run typecheck
```

If UI changed, manually validate on an Android device; automated UI tests do
not exist yet. See the **Android release build & verify** section below for
the standalone-APK workflow that survives USB disconnect.

## Android release build & verify (standalone APK)

Standard incremental rebuild — Gradle picks up JS/RN changes automatically
through `createBundleReleaseJsAndAssets` once the source tree changes:

```bash
cd android && ./gradlew assembleRelease
adb install -r android/app/build/outputs/apk/release/app-release.apk
```

The APK is **standalone**: it embeds the JS bundle (`assets/index.android.bundle`,
Hermes bytecode) so the app keeps working after USB disconnect — no Metro
dependency. Do **not** use `npx expo run:android --variant release` as the only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Helldez/JarvisQ](https://github.com/Helldez/JarvisQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
