---
trigger: always_on
description: i3X Explorer is a cross-platform desktop application for browsing and monitoring I3X (Industrial Information Interface eXchange) API servers. Similar to MQTT Explorer but for the I3X protocol.
---

# i3X Explorer Project

## Overview

i3X Explorer is a cross-platform desktop application for browsing and monitoring I3X (Industrial Information Interface eXchange) API servers. Similar to MQTT Explorer but for the I3X protocol.

**Stack:** Electron + React + TypeScript + Vite + Tailwind CSS

## Project Structure

```
i3x-explorer/
├── electron/                # Electron main process
│   ├── main.ts             # App entry, window management
│   └── preload.cjs         # Context bridge for IPC (CommonJS, loaded by Electron)
├── src/                    # React renderer
│   ├── main.tsx            # React entry
│   ├── App.tsx             # Root component
│   ├── api/                # I3X API client
│   │   ├── client.ts       # HTTP client (fetch-based)
│   │   ├── types.ts        # TypeScript interfaces
│   │   └── subscription.ts # SSE subscription handler
│   ├── components/         # UI components
│   │   ├── layout/         # Toolbar, Sidebar, MainPanel, BottomPanel
│   │   ├── tree/           # TreeView for hierarchy browsing
│   │   ├── details/        # Detail panels (Namespace, ObjectType, Object)
│   │   ├── connection/     # ConnectionDialog
│   │   └── subscriptions/  # SubscriptionPanel
│   ├── stores/             # Zustand state management
│   │   ├── connection.ts   # Server connection state
│   │   ├── explorer.ts     # Tree/selection state
│   │   └── subscriptions.ts# Active subscriptions & live values
│   └── styles/             # Tailwind CSS
├── build/                  # Build resources (icons, entitlements)
├── scripts/                # Build helper scripts
├── release/                # Built installers (not in git)
├── electron-builder.json   # Packaging configuration
├── package.json
├── vite.config.ts
└── tailwind.config.js
```

## Development

```bash
# Prerequisites: Node.js 18+ (project has .nvmrc file)
nvm use

# Install dependencies
npm install

# Run in development mode (with hot reload)
npm run dev

# Type checking
npm run typecheck
```

## Building Installers

**Important:** Use Node.js 18+ before building. The project includes an `.nvmrc` file.

```bash
# First, switch to the correct Node version
nvm use 20  # or: nvm use (if .nvmrc is configured)

# Generate icons (uses build/icon-1024.png by default)
./scripts/generate-icons.sh

# Build for all platforms (best way, recommended for releases)
./scripts/build-all.sh [mac|win|linux|all]

# Platform-specific builds
npm run build:all          # All
npm run build:mac          # macOS (Intel + Apple Silicon)
npm run build:mac:x64      # macOS Intel only
npm run build:mac:arm64    # macOS Apple Silicon only
npm run build:win          # Windows (x64 + x86 + portable)
npm run build:linux        # Linux (AppImage + tar.gz)
```

### macOS Notarization

For notarized macOS builds (required to avoid "app is damaged" on Apple Silicon downloads), create `scripts/set-apple-vars.sh` (git-ignored) with:

```bash
export APPLE_ID="you@example.com"
export APPLE_APP_SPECIFIC_PASSWORD="xxxx-xxxx-xxxx-xxxx"  # from appleid.apple.com
export APPLE_TEAM_ID="XXXXXXXXXX"                          # from developer.apple.com/account
```

Also requires a **Developer ID Application** certificate (not "Mac Installer Distribution") in the keychain — create via Xcode → Settings → Accounts → Manage Certificates.

`build-all.sh` sources this file automatically. If absent or vars unset, the build completes unsigned with a warning. Notarization logic lives in `scripts/notarize.cjs` (afterSign hook).

### Windows Signing

Windows signing uses **Azure Trusted Signing** (Microsoft-managed CA, ~$10/month). It fully suppresses the SmartScreen "Unknown publisher" warning. Signing must be done on a Windows machine — `signtool.exe` (a Windows-only binary) does the Authenticode embedding; there is no viable cross-platform path for this step.

**To sign a Windows build**, run on a Windows box:
```powershell
.\scripts\build-sign-win.ps1
```

The script builds the installer, auto-downloads the Azure Trusted Signing dlib from NuGet on first run (cached in `scripts\.azure-signing\`, git-ignored), then signs all `.exe` files.

Credentials go in `scripts\set-azure-vars.ps1` (git-ignored; template at `scripts\set-azure-vars.example.ps1`):
```powershell
$env:AZURE_TENANT_ID                = "..."   # Entra ID → Overview → Tenant ID
$env:AZURE_CLIENT_ID                = "..."   # Entra ID → App registrations → your app → Application (client) ID
$env:AZURE_CLIENT_SECRET            = "..."   # same app → Certificates & secrets → Client secrets → Value
$env:AZURE_TRUSTED_SIGNING_ENDPOINT = "..."   # Trusted Signing account → Overview → URI
$env:AZURE_TRUSTED_SIGNING_ACCOUNT  = "..."   # Trusted Signing account → Overview → Name
$env:AZURE_TRUSTED_SIGNING_PROFILE  = "..."   # Trusted Signing account → Certificate profiles → profile name
```

The app registration needs the **Trusted Signing Certificate Profile Signer** role assigned on the signing account (Azure portal → signing account → Access control (IAM)).

Full setup walkthrough: see `WINDOWS-SIGNING.md`.

**Output:** `release/{version}/`

| Platform | Artifacts |
|----------|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ace-technologies-inc/i3X-Explorer](https://github.com/ace-technologies-inc/i3X-Explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
