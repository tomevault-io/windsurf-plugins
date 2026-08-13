---
trigger: always_on
description: Cross-platform TikTok/Douyin no-watermark downloader. Four clients share one Python core:
---

# AGENTS.md

## What this is

Cross-platform TikTok/Douyin no-watermark downloader. Four clients share one Python core:
- **Python CLI** (`python/`) — the installable `media-downloader` package, also used by Windows GUI and WebUI
- **iOS app** (`apps/ios/`) — native SwiftUI, uses shared Swift library from `apple/`
- **macOS app** (`apps/macos/`) — native SwiftUI, same shared Swift library
- **Windows GUI** (`apps/windows/`) — Python + tkinter (sv-ttk), wraps the Python core

## Repo layout

```
python/                  # Python package (source of truth for version + logic)
  src/media_downloader/  # Package source
    __init__.py          # __version__ (synced from version.json)
    cli.py               # Entry points: main(), douyin_main(), tiktok_main()
    core/                # Models, downloader dispatch, policies, network
    platforms/           # douyin.py, tiktok.py — platform-specific parsers + downloaders
    browser/             # Playwright wrapper with stealth
    i18n/                # Internationalization catalogs
  tests/                 # Pytest unit tests (no network, no browser)
  pyproject.toml         # Package metadata, deps, pytest config
  uv.lock                # Lockfile (uv package manager)
apple/                   # Shared Swift library (MediaDownloaderCore) for iOS + macOS
  Package.swift          # SPM package, platforms: iOS 17+, macOS 14+
  Sources/MediaDownloaderCore/  # ShareTextParser, Models, NativeMediaScraper, MediaDownloadService
apps/
  ios/                   # Xcode project, SwiftUI app
  macos/                 # Xcode project, SwiftUI app
  android/               # Android app (Kotlin, Gradle; Douyin downloader, versioned 0.1.x independently)
  windows/               # Python GUI (gui/) + Inno Setup installer (installer/)
  web/                   # Gradio WebUI (single file webui.py)
scripts/                 # Build entry points (see Build section)
Casks/                   # Homebrew cask (tiktok-douyin-dl.rb) — sha256 updated by CI
skills/                  # AgentSkills-compatible media-downloader skill
docs/                    # Architecture, release notes, policy docs
  MAINTENANCE.md          # ⭐ 维护指南：客户端全景 / 版本号 / 攒更新工作流 / 发版 / 策略（改任何端前先读）
  architecture.md        # Architecture overview
  version-policy.md      # version-policy.json spec
  download-policy.md     # download-policy.json spec
  releases/              # Per-release download pages (used as GitHub Release notes)
version-policy.json      # Client-side version nag/block rules (fail-open)
download-policy.json     # Client-side download enable/disable (fail-closed)
changelog.json           # Machine-readable per-platform changelog (generated from CHANGELOG.md)
version.json             # SINGLE source of truth for all version numbers (scripts/sync-versions.py)
```

## Platforms

Clients: Windows GUI, macOS/iOS native apps, Android (Kotlin), CLI (Windows/Linux/macOS), WebUI.
The shared policy/changelog platform keys are: `cli`, `windows`, `macos`, `ios`, `android` (plus `all` for changelog).
Android versions (0.1.x) are independent of the `media_downloader.__version__` line.

## Commands

### Python development

```bash
# Setup (from repo root)
cd python && uv sync                    # Install deps into .venv
uv run playwright install chromium      # Required for browser-based downloads

# Run tests
cd python && uv run pytest              # All tests
cd python && uv run pytest tests/test_cli.py           # Single file
cd python && uv run pytest tests/test_cli.py::test_detect_platform  # Single test
cd python && uv run pytest -k "version_policy"          # By keyword

# Run CLI from source
cd python && uv run media-downloader "share text or link"
cd python && uv run python -m media_downloader.cli "share text or link"
```

### Build scripts (run from repo root)

```bash
./scripts/build-apple.sh all            # Build unsigned iOS IPA + macOS DMG
./scripts/build-apple.sh ios            # iOS only
./scripts/build-apple.sh macos          # macOS only
./scripts/build-linux.sh                # Linux CLI (PyInstaller one-file)
./scripts/build-macos-cli.sh            # macOS CLI for host arch (arm64/x86_64, PyInstaller one-file + zip)
./scripts/build-windows.ps1             # Windows (PowerShell, needs Inno Setup)
./scripts/update-changelog-json.py      # Regenerate changelog.json from CHANGELOG.md (run at release)
./scripts/sync-versions.py               # Propagate version.json to all hard-coded version constants
./scripts/sync-versions.py --policies    # Also mirror policy min_version fields
./scripts/release.sh                    # Bump policy timestamps, tag, push → CI builds all
```

Apple build env vars: `APPLE_VERSION`, `APPLE_BUILD_NUMBER`, `APPLE_OUTPUT_DIR`, `APPLE_SIGNING_IDENTITY`, `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID`.

### Release flow

`./scripts/release.sh` does NOT build locally. It:
1. Reads version from `media_downloader.__version__`
2. Verifies the tag doesn't exist and working tree is clean
3. Bumps `updated_at` in `version-policy.json` and `download-policy.json`
4. Pushes branch + tag → `.github/workflows/release.yml` builds all platforms on GitHub runners

## Version management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Francis-Xavier-code/tiktok-douyin-dl](https://github.com/Francis-Xavier-code/tiktok-douyin-dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
