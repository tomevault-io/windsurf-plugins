---
trigger: always_on
description: Native iOS Home Assistant dashboard app. Renders HA Lovelace dashboards natively across iOS 9.3.5 (iPad 2, armv7) through iOS 18+ (iPhone 16 Pro Max, arm64), providing a kiosk-friendly wall-mounted display experience on old iPads while also working as a mobile dashboard on modern devices.
---

# HA Dashboard

Native iOS Home Assistant dashboard app. Renders HA Lovelace dashboards natively across iOS 9.3.5 (iPad 2, armv7) through iOS 18+ (iPhone 16 Pro Max, arm64), providing a kiosk-friendly wall-mounted display experience on old iPads while also working as a mobile dashboard on modern devices.

## Published Links

- **App Store**: https://apps.apple.com/gb/app/ha-dash/id6759347912
- **Landing Page**: https://ha-dashboard.github.io/ios-app/
- **Support Page**: https://ha-dashboard.github.io/ios-app/support.html
- **Privacy Policy**: https://ha-dashboard.github.io/ios-app/privacy.html
- **GitHub**: https://github.com/ha-dashboard/ios-app

## Project Goals

- Render the user's HA Lovelace dashboard natively — not a web view
- Visual parity across all devices: same cards, same layout, same data
- Kiosk mode for wall-mounted iPads (hide nav bar, prevent sleep, triple-tap escape)
- Fast startup and smooth scrolling, especially on the iPad 2's A5 chip (512MB RAM)

## Build Setup

Two Xcode versions are required:

| Xcode | Path | Purpose |
|-------|------|---------|
| **13.2.1** | `/Applications/Xcode-13.2.1.app` | Provides armv7 SDK stubs for linking the universal device build (iPad 2). |
| **26** | `/Applications/Xcode.app` | Builds all targets: arm64 sim, x86_64 legacy sim (RosettaSim), and device. |

### Build Targets

| Target | Command | Arch | iOS Min | SDK | Notes |
|--------|---------|------|---------|-----|-------|
| Simulator | `scripts/build.sh sim` | arm64 | 15.0 | Xcode 26 | Native arm64 sim for iOS 16+ |
| RosettaSim | `scripts/build.sh rosettasim` | x86_64 | 9.0 | Xcode 26 | Legacy sim for iOS 9–14 via RosettaSim. Uses `MERGED_BINARY_TYPE=none` to disable mergeable libraries — the default Debug stub+dylib pattern crashes on legacy runtimes' libdispatch. |
| Device | `scripts/build.sh device` | armv7+arm64 | 9.0 | Xcode 26 clang + Xcode 13 link stubs | Universal binary. armv7 compiled with Xcode 26 clang, linked against Xcode 13 SDK. arm64 via xcodebuild. |

- **XcodeGen** generates `HADashboard.xcodeproj` from `project.yml` — run `scripts/regen.sh` after changing project.yml
- `regen.sh` sources `.env` to inject your Team ID and Bundle ID into the project before generation
- **Signing**: Automatic provisioning via App Store Connect API key (credentials in `.env`)

## Environment Configuration

All secrets and device-specific configuration live in `.env` at project root (git-ignored). Copy `.env.example` to get started:

```bash
cp .env.example .env
# Then fill in your values
```

Key variables:
- `APPLE_TEAM_ID`, `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_KEY_PATH` — Apple signing
- `BUNDLE_ID` — your app bundle identifier (used by build + deploy scripts)
- `HA_SERVER`, `HA_TOKEN`, `HA_DASHBOARD` — Home Assistant connection
- Device UDIDs for physical deploy targets (see `.env.example` for full list)
- Simulator UDIDs are auto-detected by device name if not set

**Never commit `.env`, `private_keys/`, or any tokens/passwords to source control.**

## Deployment Targets

| Device | Arch | iOS | Deploy Method |
|--------|------|-----|---------------|
| iPad 2 | armv7 | 9.3.5 | WiFi SSH (jailbroken) or Unraid USB |
| iPad 3 | armv7 | 9.3.5 | WiFi SSH (jailbroken) |
| iPad Mini 4 | arm64 | 15.x | WiFi via ios-deploy + pymobiledevice3 |
| iPad Mini 5 | arm64 | 26.x | WiFi via devicectl |
| iPhone 16 Pro Max | arm64 | 18.x | WiFi via devicectl |
| iPad Simulator | arm64 | 16.4+ | `xcrun simctl install/launch` |
| iPhone Simulator | arm64 | 16.4+ | `xcrun simctl install/launch` |
| Legacy Simulator | x86_64 | 9.3–14.x | `rosettasim-ctl install/launch` (via RosettaSim) |

## Versioning & Release

Version is derived from **git tags** — no files to edit for a version bump.

- `scripts/build.sh` reads the latest `v*` tag → `MARKETING_VERSION`, commit count → `CURRENT_PROJECT_VERSION`
- `Info.plist` uses `$(MARKETING_VERSION)` / `$(CURRENT_PROJECT_VERSION)` build setting variables
- `project.yml` has `0.0.0` / `0` fallback defaults (only used if building directly from Xcode without the build script)
- **Never hardcode version numbers** in Info.plist, project.yml, or pbxproj

### Release Workflow

To cut a release:

```bash
# 1. Tag HEAD (annotated tag)
git tag -a v1.2.1 -m "v1.2.1: summary of changes"
git push origin main --tags

# 2. Create GitHub release with notes
gh release create v1.2.1 --title "v1.2.1" --latest --notes "..."
```

To **retag** (move an existing tag to current HEAD):

```bash
git tag -d v1.2.1                    # Delete local tag
git push origin :refs/tags/v1.2.1    # Delete remote tag
git tag -a v1.2.1 -m "..."           # Recreate at HEAD
git push origin v1.2.1               # Push new tag
# Then delete old GitHub release and create a new one
gh release delete v1.2.1 --yes
gh release create v1.2.1 --title "v1.2.1" --latest --notes "..."
```

### CI Pipeline (`.github/workflows/build.yml`)

Triggered on pushes to `main` and `v*` tags:

| Job | Trigger | What it does |
|-----|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ha-dashboard/ios-app](https://github.com/ha-dashboard/ios-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
