---
trigger: always_on
description: Android plugin for exteraGram (Telegram fork) loaded at runtime via DEX injection. Two deliverable artifacts: `classes.dex` (the plugin) and `loader.plugin` (the Python loader that downloads/loads the DEX).
---

# re:extera — Agent Guide

## What this is

Android plugin for exteraGram (Telegram fork) loaded at runtime via DEX injection. Two deliverable artifacts: `classes.dex` (the plugin) and `loader.plugin` (the Python loader that downloads/loads the DEX).

## Build commands (exact)

```bash
# Build the DEX plugin (assembleRelease AAR → d8 → classes.dex)
./gradlew buildDex

# Build the Python loader plugin (concatenates loader/*.py → loader.plugin)
python3 loader/build.py
```

**Requirements**: JDK 17, Android SDK (compileSdk 35, build-tools 36.0.0), Python 3.x.

**Output**: `build/dex/classes.dex` and `build/plugin/loader.plugin`.

## Project structure

```
re-extera/
├── build.gradle                  # Android library build script
├── libs/exteragram.jar           # compileOnly dependency (exteraGram SDK stubs)
├── loader/                       # Python loader (concatenated by build.py)
│   ├── build.py                  # Concatenation and syntax-checking script
│   ├── config.py                 # Stores cached versions and rate-limiting data
│   ├── dex.py                    # GitHub releases fetching and DEX loading engine
│   ├── plugin.py                 # Main exteraGram BasePlugin implementation & UI dialogs
│   ├── metadata.py               # Plugin metadata (__version__, __id__, __min_version__)
│   └── (utils.py, constants.py, imports.py)
└── src/main/java/ni/shikatu/re_extera/
    ├── Main.java                 # Entry point: initAndStart() → DB init & hooks
    ├── Defaults.java             # Constants for Ghost mode (typing, reading, etc.)
    ├── db/                       # Custom SQLite implementation (re_extera.db)
    │   ├── ReExteraDb.java       # Database helper and CRUD operations (HandlerThread)
    │   └── (Entities: DialogExclusion, ShadowbanEntry)
    ├── hooks/                    # ~50+ Xposed hooks across Telegram classes
    │   ├── HookInit.java         # Central hook registry via XposedBridge
    │   ├── chatactivity/         # Chat UI hooks (menus, message processing)
    │   ├── chatmessagecell/      # Message cell hooks (deleted message transparency)
    │   ├── connectionsmanager/   # Network hooks (Ghost mode sendRequestInternal interception)
    │   ├── dialogsactivity/      # Dialog list hooks
    │   ├── messagescontroller/   # Core logic hooks (filtering, shadowbans, update loop)
    │   ├── messagesstorage/      # SQLite hooks (intercepting markMessagesAsDeleted)
    │   └── (other hook subpackages: notificationmanager, profileactivity, sendmessageshelper, etc.)
    ├── localization/
    │   └── Localization.java     # Translations for DEX settings
    ├── settings/
    │   ├── Settings.java         # SharedPreferences abstraction ("re_extera")
    │   └── newui/                # Settings screen fragments (GhostFragment, CustomizationFragment, etc.)
    ├── ui/                       # Additional UI components
    │   ├── DeletedMessagesInChatFragment.java
    │   ├── RegexFiltersFragment.java
    │   └── (ShadowbanDialog, ExclusionsFragment)
    └── utils/                    # 14 utility classes
        ├── MessageForwarder.java # Logic for redirecting deleted/edited messages to Saved Messages
        ├── ReflectionUtils.java  # Reflection helpers for accessing obfuscated Telegram fields
        ├── GhostMenuHelper.java  # Helper for long-press ghost mode menus
        └── (AccountUtils, DrawableUtils, ExclusionUtils, ShadowbanCache, etc.)
```

## Architecture

- **Entry point**: `ni.shikatu.re_extera.Main.initAndStart()` — called by the Python loader after DEX injection
- **Hooking**: Uses `de.robv.android.xposed.XposedBridge` to hook ~50+ exteraGram/Telegram methods at runtime
- **Settings**: `SharedPreferences("re_extera")` — boolean/int/float/string key-value store
- **Database**: Custom SQLite (`re_extera.db`, version 11) with 7 tables: `deleted_keys`, `message_edits`, `exception_users`, `regex_filters`, `shadowban_users`, `read_events`, `last_online_users`. All writes go through a dedicated HandlerThread.
- **Ghost mode**: Intercepts `ConnectionsManager.sendRequestInternal` to block typing/reading/online/stories requests. Request types defined in `Defaults.java`.
- **Versioning**: Auto-generated from git tags. Tag format `v<plugin_ver>-<tg_ver>` (e.g. `v2.8.3-12.9.0`). Dev builds use `{yyyyMMddHHmmss}-{commit}`.

## CI & releasing

- GitHub Actions on push to master/main or `v*` tags
- Artifacts: `classes.dex` + `loader.plugin` per commit (dev artifacts on nightly.link)
- Release tags: `v<plugin_ver>-<tg_ver>` → release named `v<plugin_ver> for <tg_ver>`
- Loader channels: "Release" (stable GitHub releases) and "Dev" (latest CI artifact)

## Loader behavior

- The Python loader (`loader.plugin`) runs inside exteraGram's plugin engine
- On load: checks local path → cache → downloads fresh DEX
- DEX loading: tries `InMemoryDexClassLoader` first, falls back to `DexClassLoader` from file
- Update checks rate-limited (60s cooldown)
- Min exteraGram version: `12.8.1` (from `loader/metadata.py`)
- Plugin metadata: `__id__ = "re_extera_loader"`, `__version__ = "2.8.3"`

## Hooks troubleshooting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fossSquad/re-extera](https://github.com/fossSquad/re-extera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
