---
trigger: always_on
description: A native Android application (`com.mrj.fancyai`) that embeds a **WebView-based virtual phone OS** via a thick HTML/CSS/JS frontend loaded from assets. The app functions as an AI companion platform with messaging, image generation, social feed, games, and character management — all running client-side inside the WebView.
---

# AGENTS.md — Fancy AI

## Project Overview

A native Android application (`com.mrj.fancyai`) that embeds a **WebView-based virtual phone OS** via a thick HTML/CSS/JS frontend loaded from assets. The app functions as an AI companion platform with messaging, image generation, social feed, games, and character management — all running client-side inside the WebView.

---

## Build & Run

| Command | Purpose |
|---|---|
| `./gradlew assembleDebug` | Build debug APK |
| `./gradlew installDebug` | Install on connected device |
| `./gradlew assembleRelease` | Build release APK |
| `./gradlew test` | Run unit tests (currently none) |
| `./gradlew connectedAndroidTest` | Run instrumented tests (currently none) |

- **AGP**: `8.13.2` | **compileSdk/targetSdk**: `36` | **minSdk**: `24`
- **Java Version**: `17` (source & target compatibility)
- **Version**: `3.0.0` (versionCode: `2`)
- **Namespace**: `com.mrj.fancyai`

---

## Project Structure

```
app/src/main/
  java/com/mrj/fancyai/
    MainActivity.java         # Single Activity — hosts WebView with JS bridge
  assets/
    index.html                # Virtual OS shell — home screen + app launcher
    js/
      core/
        api.js                # LLM communication layer (DeepInfra/OpenRouter/Custom)
        db.js                 # Media storage (Android Native Disk via Bridge, with localStorage fallback)
        state.js              # Global state (characters, settings, sessions, social feeds)
      apps/
        contacts.js           # Character manager (create/edit/delete identities)
        gallery.js            # Stored image gallery from IndexedDB
        games.js              # Games hub (Adventure, RPG, Hacking, etc.)
        imaging.js            # Image generation engine (Forge/Local Dream)
        messenger.js          # Chat interface per character
        rebbit.js             # NSFW amateur feed — bots post explicit photos (subreddit categories, ImageDB storage)
        settings.js           # LLM config, API keys, system prompts, user profile, backup/restore
        ustagram.js           # SFW lifestyle social feed — bots post photorealistic Instagram-style photos
        y.js                  # Text-only micro-blog feed — bots post statuses with threaded replies
      lib/
        jszip.min.js          # JSZip for backup export/import
    res/
      layout/activity_main.xml   # WebView layout
      xml/
        network_security_config.xml   # Cleartext for 127.0.0.1/localhost/10.0.2.2
        file_paths.xml              # FileProvider cache paths
      values/                 # strings.xml, colors.xml, themes.xml
      values-night/           # Dark theme
      drawable/               # Launcher icons
      mipmap-*/               # App launcher icons
  AndroidManifest.xml         # Permissions: INTERNET, storage, media access
build.gradle.kts              # Module-level build config
```

Root-level files `styles.css` and `script.js` are JetBrains build report viewer files — not part of the app.

---

## Architecture & Data Flow

### OS Shell (`index.html`)

The `OS` global object manages app lifecycle — `OS.launch('AppName', params)` hides the home screen, shows the app window, and calls `window[AppName].init(slot, params)`. The `goHome()` method resets the UI. A `popstate` listener handles back navigation (with a special case for the ImagingApp lightbox).

Key `OS` methods:
- `launch(appName, params)` — switches apps, pushes history state, calls `cleanup()` on previous app if it implements it
- `goHome()` — returns to home screen, calls `cleanup()` on current app
- `goBack()` — pops in-app nav stack (`OS.navStack`) or goes home
- `pushView(restoreFn)` — pushes an in-app view for custom back navigation
- `formatMarkdown(text)` — renders `**bold**`, `*italic*`, `***bold italic***` to HTML

Apps can optionally implement a `cleanup()` method to stop timers/intervals when the OS switches away from them.

### Core Modules

| Module | File | Role |
|---|---|---|
| `State` | `core/state.js` | Global singleton — holds `characters[]`, `settings{}` (with `systemPrompts[]`/`activePromptId`), `sessions{}`, `userProfile{}`, `instagramPosts[]`, `redditPosts[]`, `xPosts[]`, `activeCharId`, `maxSessionMessages`. Persisted to Android native file `state.json` via `AndroidBridge.saveToFile`, with `localStorage` key `fancy_ai_state` as fallback. |
| `API` | `core/api.js` | LLM communication — supports DeepInfra (`api.deepinfra.com`), OpenRouter (`openrouter.ai`), and custom OpenAI-compatible endpoints. Streaming support via `ReadableStream`. Injects role directives based on context (`chat`, `social`, `game`). Uses `systemPrompts`/`activePromptId` for global guidance. Image generation triggered via `flux prompt:` in response text (not JSON tool calls). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mr-J-369/Fancy-Ai](https://github.com/Mr-J-369/Fancy-Ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
