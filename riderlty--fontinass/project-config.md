---
trigger: always_on
description: FontInAss is a tool that **subsets fonts and embeds them into ASS/SSA/SRT subtitle files** in real-time. It solves the problem of subtitles not displaying correctly on devices that lack the required fonts by embedding a minimal subset of each font directly into the subtitle file using UUEncode.
---

# CLAUDE.md — FontInAss

## Project Overview

FontInAss is a tool that **subsets fonts and embeds them into ASS/SSA/SRT subtitle files** in real-time. It solves the problem of subtitles not displaying correctly on devices that lack the required fonts by embedding a minimal subset of each font directly into the subtitle file using UUEncode.

Designed to work as a **transparent proxy** for Emby/Jellyfin media servers — intercepts subtitle requests, processes them with embedded fonts, and returns the modified subtitle to the client.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Nginx (port 8012) - reverse proxy for Emby/Jellyfin   │
│  Intercepts /videos/*/Subtitles/* requests              │
│  Proxies subtitle requests → Python service (8011)      │
│  Proxies all other requests → Emby server               │
│  Proxies JS files → Python for web font rendering hack  │
└─────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────┐
│  Python FastAPI service (port 8011)                     │
│  ┌─────────────┐ ┌──────────────┐ ┌──────────────────┐ │
│  │ SubSetter   │ │ FontManager  │ │ dirmonitor       │ │
│  │ (subset +   │ │ (DB + cache  │ │ (watchdog on     │ │
│  │  embed)     │ │  + download) │ │  font dirs)      │ │
│  └──────┬──────┘ └──────┬───────┘ └────────┬─────────┘ │
│         │               │                  │            │
│  ┌──────▼───────────────▼──────────────────▼─────────┐  │
│  │          SQLite DB (localFonts.ver.2.6.db)        │  │
│  │          + TTL/LRU Caches (font + subtitle)       │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Cython/C++ extension (py2cy/c_utils)             │   │
│  │ - uuencode: UU-encode font binary for ASS embed  │   │
│  │ - analyseAss: Parse ASS to extract font→unicode   │   │
│  │ - parse_table: Parse OS/2 and head font tables   │   │
│  └──────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Vue 3 Frontend (/subset) - Batch subtitle tool   │   │
│  │ Vite + Ant Design Vue + vue-i18n                  │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## Key Commands

### Build Cython Extension (required first)
```bash
python src/py2cy/setup.py
```

### Build Frontend
```bash
cd src/subset && npm install && npm run build
```

### Install Python Dependencies
```bash
# Using uv (recommended)
uv sync

# Using pip
pip install -r requirements.txt
```

### Run Locally
```bash
export EMBY_SERVER_URL="http://[ip]:[port]"
python src/main.py

# Or with uv
uv run src/main.py
```

### Run with Docker
```bash
docker run -d --name=fontinass --restart=unless-stopped \
  -p 8011:8011 -p 8012:8012 \
  -e EMBY_SERVER_URL=http://[ip]:[port] \
  -v /yourDir/fontinass/data:/data \
  -v /yourDir/fontinass/fonts:/fonts \
  riderlty/fontinass:latest
```

### Docker Build
```bash
# One-step build (all stages in Dockerfile)
docker build . -f Dockerfile -t riderlty/fontinass:latest

# Multi-stage: build builder image first
docker build . -f Dockerfile-builder -t riderlty/fontinass-builder:latest
# Then build noproxy
docker build . -f Dockerfile-noproxy -t riderlty/fontinass:noproxy
# Then build default (adds nginx)
docker build . -f Dockerfile-default -t riderlty/fontinass:latest
```

## Project Structure

```
fontInAss/
├── src/
│   ├── main.py              # FastAPI app entry point, HTTP endpoints
│   ├── constants.py          # All env vars, paths, config constants
│   ├── fontmanager.py        # Font DB (SQLAlchemy/SQLite), font loading, online download
│   ├── subsetter.py          # Core logic: ASS analysis → font subset → embed
│   ├── utils.py              # SRT→ASS conversion, font scoring, ASS manipulation
│   ├── analyseAss.py         # Pure Python ASS parser (backup, mostly replaced by C++)
│   ├── dirmonitor.py         # Watchdog-based font directory monitor
│   ├── colorAdjust.py        # HSV color adjustment for HDR subtitles
│   ├── logs.py               # Async log file manager for missing font/glyph logs
│   ├── docker.init.py        # Docker entrypoint: generates nginx config from env vars
│   ├── create_onlineFonts.json.py  # Utility to generate custom font online DB
│   ├── html/color.html       # Web UI for color/brightness adjustment
│   ├── py2cy/                # Cython/C++ performance-critical code
│   │   ├── c_utils.pyx       # uuencode, analyseAss (C++ impl), parse_table
│   │   ├── cpp_utils.cpp     # C++ ASS analysis implementation
│   │   └── setup.py          # Cython build script
│   └── subset/               # Vue 3 frontend for batch subtitle processing
│       ├── package.json      # Node.js dependencies (vue, ant-design-vue, vite)
│       ├── vite.config.js    # Vite config
│       └── src/
│           ├── main.js       # Vue app with i18n (zh-CN, en-US)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RiderLty/fontInAss](https://github.com/RiderLty/fontInAss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
