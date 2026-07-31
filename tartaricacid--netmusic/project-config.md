---
trigger: always_on
description: **Generated:** 2026-04-26 01:08:50 +08:00
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-26 01:08:50 +08:00
**Commit:** 671a9c7
**Branch:** 1.20

## Language policy

- 默认使用简体中文回答。
- 除非我明确要求英文，否则不要切换英文叙述。
- 代码、命令、报错、API 名称保持原文，不要强行翻译。
- 提问澄清时也使用中文。

## OVERVIEW

Forge 1.20.1 port of Net Music Mod. Java 17, Parchment mappings, shaded audio codecs, optional compat layers for Cloth
Config, Sophisticated Backpacks, and Touhou Little Maid.

## STRUCTURE

```text
./
├── src/main/java/com/github/tartaricacid/netmusic/
│   ├── api/           # NetEase HTTP/API wrappers, lyric/search/POJO parsing
│   ├── client/        # client-only audio, GUI, renderers, layer registration
│   ├── compat/        # optional integrations grouped by external mod
│   ├── config/        # Forge config spec + music list persistence
│   ├── event/         # server-side startup hooks
│   ├── init/          # deferred registers and common lifecycle hooks
│   ├── network/       # packet channel and message types
│   └── NetMusic.java  # @Mod entrypoint
├── src/main/resources/
│   ├── META-INF/      # mods.toml, access transformer, audio SPI service files
│   ├── assets/netmusic/
│   ├── data/netmusic/
│   └── licenses/      # bundled third-party license texts
├── src/generated/resources/  # configured as resources input, currently empty
└── .github/workflows/        # single Gradle build/release workflow
```

## WHERE TO LOOK

| Task                        | Location                                                                     | Notes                                                       |
|-----------------------------|------------------------------------------------------------------------------|-------------------------------------------------------------|
| Mod bootstrap               | `src/main/java/com/github/tartaricacid/netmusic/NetMusic.java`               | Registers blocks/items/menus/sounds/config and early compat |
| Common setup                | `src/main/java/com/github/tartaricacid/netmusic/init/CommonRegistry.java`    | `FMLCommonSetupEvent` enqueues `NetworkHandler::init`       |
| Server startup data load    | `src/main/java/com/github/tartaricacid/netmusic/event/ServerEvent.java`      | Loads `music.json`/config songs on dedicated server start   |
| Forge config                | `src/main/java/com/github/tartaricacid/netmusic/config/GeneralConfig.java`   | Common config spec, lyrics/proxy/backpack toggles           |
| Music list persistence      | `src/main/java/com/github/tartaricacid/netmusic/config/MusicListManage.java` | Reads/writes `config/net_music/music.json`                  |
| Packets                     | `src/main/java/com/github/tartaricacid/netmusic/network/NetworkHandler.java` | Core channel + base packet ids                              |
| Client setup                | `src/main/java/com/github/tartaricacid/netmusic/client/init/`                | GUI screens, block entity renderer, model layers            |
| Audio runtime               | `src/main/java/com/github/tartaricacid/netmusic/client/audio/`               | Highest-risk subsystem; custom streams and codec providers  |
| Mod integrations            | `src/main/java/com/github/tartaricacid/netmusic/compat/`                     | Cloth, Sophisticated Backpacks, TLM                         |
| Mod metadata                | `src/main/resources/META-INF/mods.toml`                                      | Mod id, dependency ranges, version token expansion          |
| Audio service loader wiring | `src/main/resources/META-INF/services/`                                      | Registers audio readers and format converters               |
| Default assets              | `src/main/resources/assets/netmusic/`                                        | Lang, models, textures, sounds, bundled `music.json`        |
| Datapack content            | `src/main/resources/data/netmusic/`                                          | Recipes and loot tables                                     |

## CODE MAP

| Symbol                                             | Type        | Location                                                                             | Refs | Role                                                             |
|----------------------------------------------------|-------------|--------------------------------------------------------------------------------------|------|------------------------------------------------------------------|
| `NetMusic`                                         | class       | `src/main/java/com/github/tartaricacid/netmusic/NetMusic.java`                       | -    | `@Mod` entrypoint                                                |
| `NetMusic()`                                       | constructor | same                                                                                 | -    | Initializes API, registers deferred registers, registers config  |
| `NetworkHandler`                                   | class       | `src/main/java/com/github/tartaricacid/netmusic/network/NetworkHandler.java`         | -    | Core packet channel                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TartaricAcid/NetMusic](https://github.com/TartaricAcid/NetMusic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
