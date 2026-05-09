---
trigger: always_on
description: OpenDraft is the **open-source core** (MIT license). A separate **OpenDraft-Pro** repo (private, at `../OpenDraft-Pro/`) extends it with commercial plugins.
---

# OpenDraft — Claude Instructions

## Open-Core Architecture

OpenDraft is the **open-source core** (MIT license). A separate **OpenDraft-Pro** repo (private, at `../OpenDraft-Pro/`) extends it with commercial plugins.

### Plugin System

The core exposes a plugin architecture so Pro features can be added without modifying core code:

- **Frontend:** `frontend/src/plugins/registry.ts` — `pluginRegistry.register()` to add menu items, sidebar panels, routes, and editor extensions
- **Backend:** `backend/app/plugins.py` — `register_router()` and `register_hook()` for API routes and lifecycle hooks
- **Integration points:** MenuBar appends plugin menu items, App.tsx renders plugin routes, ScreenplayEditor renders plugin panels and editor extensions

### Key Rules

- **Never add commercial/Pro features to this repo** — they go in OpenDraft-Pro
- Plugin architecture changes (registry, hooks, extension points) belong HERE
- Bug fixes to existing features go HERE — they automatically flow to Pro via git submodule
- The `pluginRegistry` import in MenuBar, App.tsx, and ScreenplayEditor is how Pro injects features at runtime

### Repo Relationship

```
OpenDraft (this repo, public, MIT)     ← upstream
    ↑
OpenDraft-Pro (private, proprietary)   ← imports this as git submodule at core/
```

---

## macOS Desktop Build — Code Signing & Notarization

The macOS `.dmg` is built via **GitHub Actions** (in `.github/workflows/release.yml`), same as Windows, Linux, and Android. It is signed and notarized automatically when the correct GitHub secrets are configured.

### GitHub Secrets for macOS

| Secret | Description |
|--------|-------------|
| `APPLE_CERTIFICATE` | Base64-encoded `.p12` file containing the Developer ID Application certificate + private key |
| `APPLE_CERTIFICATE_PASSWORD` | Password for the `.p12` file |
| `APPLE_SIGNING_IDENTITY` | `Developer ID Application: Base Information Management Pvt. Ltd. (335RGMFDB6)` |
| `APPLE_ID` | Apple ID email for notarization |
| `APPLE_TEAM_ID` | `335RGMFDB6` |
| `APPLE_PASSWORD` | App-specific password from appleid.apple.com |

### How to export the certificate as `.p12`

1. Open Keychain Access, find `Developer ID Application: Base Information Management Pvt. Ltd.`
2. Right-click → Export Items → save as `.p12` with a password
3. Base64-encode: `base64 -i certificate.p12 | pbcopy`
4. Paste into the `APPLE_CERTIFICATE` GitHub secret

### Local build (optional)

`./build-desktop.sh` can still build a signed `.dmg` locally if you have Apple credentials in `.env`.

### Common mistakes to avoid

- **Never skip signing/notarization** — unsigned `.dmg` files trigger "damaged and can't be opened" on user machines
- Use **Developer ID Application** certificate for direct distribution (not "3rd Party Mac Developer" — that's for App Store only)
- The App Store build (`build-appstore.sh` locally, or `build-mac-appstore` CI job) uses different certificates. Don't mix them up.

## macOS App Store Build (CI)

The macOS `.pkg` for the Mac App Store is built via **GitHub Actions** (in `.github/workflows/release.yml`, the `build-mac-appstore` job). This runs **in addition to** the direct distribution `.dmg` build — both are produced on every release.

### How it works

1. Builds the Tauri app unsigned (`APPLE_SIGNING_IDENTITY="-"`)
2. Re-signs with Apple Distribution certificate and App Store entitlements
3. Embeds the macOS provisioning profile
4. Packages as `.pkg` with the Mac Installer certificate
5. Uploads to App Store Connect via API key

### GitHub Secrets for macOS App Store

Reuses `IOS_CERTIFICATE` (Apple Distribution) and `APPSTORE_API_KEY*` secrets. Additional secrets:

| Secret | Description |
|--------|-------------|
| `MAC_INSTALLER_CERTIFICATE` | Base64-encoded `.p12` of "3rd Party Mac Developer Installer" or "Mac Installer Distribution" certificate |
| `MAC_INSTALLER_CERTIFICATE_PASSWORD` | Password for the `.p12` file |
| `MAC_APPSTORE_PROVISION_PROFILE` | Base64-encoded `.provisionprofile` for macOS App Store |

### Local build (optional)

`./build-appstore.sh` can still build locally if you have Apple credentials in Keychain.

## Promotion & Articles

Promotion materials (blog posts, articles, social media content) go in `Promotion/posts/` in the project root — **not** in `docs/`.

---

## Release Process

See `docs/RELEASE.md` for the full checklist. Key points:
- Use `./release.sh X.Y.Z` to automate the full release
- GitHub Actions builds **all platforms**: macOS (.dmg), Windows (.exe/.msi), Linux (.deb/.AppImage), Android (.apk/.aab), iOS (.ipa)
- macOS builds are signed and notarized via Apple secrets in GitHub
- Update "What's New" content in MenuBar.tsx and user-manual before releasing

---

## Android Build

The Android `.apk` and `.aab` are built via **GitHub Actions** (in `.github/workflows/release.yml`, the `build-android` job). There is no local Android build — the CI runner provides the SDK, NDK, and Rust cross-compilation targets.

### How it works

1. CI runs `tauri android init` to generate the Gradle project (since `src-tauri/gen/` is gitignored)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Proteus-Technologies-Private-Limited/OpenDraft](https://github.com/Proteus-Technologies-Private-Limited/OpenDraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
