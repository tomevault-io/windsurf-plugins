---
trigger: always_on
description: - **Type**: VS Code Extension with React Webview UI
---

# VS Code Scrcpy

## Overview

- **Type**: VS Code Extension with React Webview UI
- **Stack**: TypeScript 5.3, React 18, Vite 6, ESBuild, VS Code Extension API
- **Purpose**: Mirror Android device screens directly in VS Code with touch controls, file management, and ADB tools
- **Architecture**: Extension (Node.js) + Webview UI (React/browser)

This CLAUDE.md is the authoritative source for development guidelines.
Subdirectory CLAUDE.md files extend these rules with specific context.

---

## Universal Development Rules

### Code Quality (MUST)

- **MUST** write TypeScript in strict mode (enabled in both tsconfigs)
- **MUST** use 4-space indentation (Prettier config)
- **MUST** use single quotes for strings
- **MUST** include trailing commas (ES5 style)
- **MUST** run `npm run typecheck` before committing
- **MUST** run `npm run format` to ensure consistent formatting
- **MUST NOT** commit secrets, API keys, or device-specific data

### Best Practices (SHOULD)

- **SHOULD** use functional React components with hooks (no class components)
- **SHOULD** use `memo()` for components receiving callback props
- **SHOULD** co-locate related code (component + hook + styles)
- **SHOULD** use descriptive variable names (no single letters except loops/lambdas)
- **SHOULD** keep functions under 50 lines when possible
- **SHOULD** extract complex logic into service classes or hooks

### Anti-Patterns (MUST NOT)

- **MUST NOT** use `any` type without explicit justification in comments
- **MUST NOT** bypass TypeScript errors with `@ts-ignore` or `@ts-expect-error`
- **MUST NOT** use `console.log` in production code (use VS Code output channels)
- **MUST NOT** hardcode device IDs or ADB paths
- **MUST NOT** block the main thread with synchronous file operations in extension

---

## Core Commands

### Development

```bash
# Install all dependencies (root + webview-ui)
npm run install:all

# Compile everything (extension + webview)
npm run compile

# Watch mode for extension development
npm run watch

# Watch mode for webview UI development
npm run watch:webview

# Type checking
npm run typecheck

# Lint code
npm run lint

# Format code
npm run format

# Check formatting without changes
npm run format:check
```

### Building & Packaging

```bash
# Full production build (clean + typecheck + bundle + webview + VSIX)
npm run build

# Bundle extension only (minified)
npm run bundle -- --minify

# Compile webview only
npm run compile:webview

# Package VSIX for distribution
npm run package:vsix
```

### Quality Gates (run before PR)

```bash
npm run typecheck && npm run lint && npm run format:check
```

---

## Project Structure

### Extension Source (`src/`)

- **[extension.ts](src/extension.ts)** - Main entry point, command registration
- **[services/](src/services/)** - Core business logic
  - `ScrcpyService.ts` - Screen mirroring via @yume-chan/scrcpy
  - `DeviceManager.ts` - ADB device discovery and selection
  - `AdbShellService.ts` - Shell command execution
  - `DeviceInfoService.ts` - Device metadata (battery, storage, etc.)
  - `AppManager.ts` - Installed apps, launch apps
  - `DeviceFileService.ts` - File operations on device
  - `ApkInstaller.ts` - APK installation
  - `AdbLogcatService.ts` - Logcat streaming
  - `AdbPathResolver.ts` - Cross-platform ADB path detection
- **[panels/](src/panels/)** - Webview panel definitions
  - `ScrcpyPanel.ts` - Floating mirror panel
  - `FileManagerPanel.ts` - Device file browser
  - `ShellLogsPanel.ts` - ADB shell interface
  - `LogcatPanel.ts` - Logcat viewer
- **[views/](src/views/)** - Sidebar view providers
  - `ScrcpySidebarView.ts` - Main sidebar with mirror + controls

See [src/CLAUDE.md](src/CLAUDE.md) for detailed extension patterns.

### Webview UI (`webview-ui/`)

- **[src/apps/](webview-ui/src/apps/)** - Full-page applications
  - `MirrorApp.tsx` - Screen mirroring view
  - `FileManagerApp.tsx` - File browser view
  - `LogcatApp.tsx` - Logcat viewer
  - `ShellLogsApp.tsx` - Shell output viewer
- **[src/components/](webview-ui/src/components/)** - Reusable UI components
  - `VideoCanvas.tsx` - WebGL video rendering + touch handling
  - `Toolbar.tsx` - Control buttons
  - `DeviceSelector.tsx` - Device picker dropdown
  - `SettingsPanel.tsx` - Quality/FPS settings
- **[src/hooks/](webview-ui/src/hooks/)** - Custom React hooks
  - `useVideoDecoder.ts` - H.264 WebCodecs decoding
  - `useVSCodeMessages.ts` - Extension ↔ webview messaging
  - `useKeyboard.ts` - Keyboard event handling
  - `useSettingsStorage.ts` - Persistent settings
- **[src/styles/](webview-ui/src/styles/)** - CSS stylesheets (15 files)

See [webview-ui/CLAUDE.md](webview-ui/CLAUDE.md) for detailed React patterns.

### Build & Configuration

- **[esbuild.js](esbuild.js)** - Extension bundler configuration
- **[scripts/build.js](scripts/build.js)** - Full build orchestration
- **[webview-ui/vite.config.ts](webview-ui/vite.config.ts)** - Webview bundler
- **[tsconfig.json](tsconfig.json)** - Extension TypeScript config (CommonJS, Node)
- **[webview-ui/tsconfig.json](webview-ui/tsconfig.json)** - Webview TypeScript config (ESNext, React)

### Assets

- **[assets/scrcpy-server](assets/scrcpy-server)** - Scrcpy server binary (pushed to device)
- **[media/](media/)** - Compiled webview output + static HTML

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aradhya1905/vscode-scrcpy](https://github.com/Aradhya1905/vscode-scrcpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
