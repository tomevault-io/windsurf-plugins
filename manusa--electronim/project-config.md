---
trigger: always_on
description: ElectronIM is a free/libre open source Electron-based multi-instant messaging (IM) client that allows users to combine multiple messaging applications into a single browser window.
---

# ElectronIM Development Guide

ElectronIM is a free/libre open source Electron-based multi-instant messaging (IM) client that allows users to combine multiple messaging applications into a single browser window.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Requirements

- **Node.js**: v22.x (LTS) - [Download](https://nodejs.org/en/download/)
  Seems like newer Node.js versions (v24+) have some issues with Jest and native modules.

### Bootstrap and Setup
Run these commands to set up the development environment:
```bash
npm install  # Install dependencies - takes ~55 seconds
```

### Build and Bundle
- `npm run pretest` - Run linting (ESLint) and build bundles (webpack) - takes ~2 seconds
- `node webpack.js` - Build webpack bundles manually - takes ~2 seconds  
- `node webpack.js --no-lib` - Build bundles without library files for development
- `npm run build:linux` - Builds and bundles the application for Linux systems
- `npm run build:mac` - Builds and bundles the application for MacOS systems
- `npm run build:win` - Builds and bundles the application for Windows systems

### Testing
- `npm test` - Run full test suite - takes ~13 seconds, runs 1072 tests (58 test suites). NEVER CANCEL - Set timeout to 30+ minutes.
- `npm run test:e2e` - Run end-to-end tests to verify application startup - takes ~10-15 seconds
- The project uses Jest with ECMAScript modules requiring the experimental VM modules flag for Node.js

### Running the Application
- `npm run prestart && npm start` - Build and run the Electron application locally (uses `dev/settings.json` and `dev/user-data` by default for development)
- `npm start -- --user-data /path/to/userdata` - Run with a custom application data directory (useful for running multiple instances)
- `npm start -- --settings-path /path/to/settings.json` - Run with a custom settings file location (useful for testing or multiple profiles)
- `npm start -- --user-data /path/to/instance1 --settings-path /path/to/instance1/settings.json` - Run with both custom user data and settings paths
- In CI/headless environments: `DISPLAY=:99 ./node_modules/.bin/electron . --no-sandbox`
- The application requires X11 display and may need sandbox disabled in CI environments
- **Global NPX usage**: `npx electronim` - Installs and runs the latest published version from npm registry
- **Multiple instances**: Use `--user-data` to run multiple instances with separate application data (profiles, cache, sessions, etc.)

### Building Platform Packages
- `npm run build:linux` - Build Linux packages (AppImage, snap, tar.gz). NEVER CANCEL - May take 30+ minutes. Set timeout to 60+ minutes.
- `npm run build:mac` - Build macOS packages (dmg, tar.gz)  
- `npm run build:win` - Build Windows packages (zip, portable exe)
- **IMPORTANT**: Build commands fail in environments with network restrictions due to Electron header downloads (node-gyp attempting to download from https://www.electronjs.org/headers). Document this limitation if builds fail with "network connectivity" errors.

## Validation

### Pre-commit Validation
Always run these commands before committing changes:
- `npm run pretest` - Validates linting and successful bundle creation
- `npm test` - Ensures all tests pass
- `npm run test:e2e` - Validates application startup (optional, for major changes)
- The CI build (`.github/workflows/tests.yml`) will fail if linting or tests fail

### Manual Testing Scenarios
After making code changes, manually validate by:
1. **Application startup**: `npm start` - Should open the main window with tabs for configured services
2. **Settings configuration**: Open settings (first launch or menu), add messaging service URLs:
   - WhatsApp Web: `https://web.whatsapp.com`
   - Telegram Web: `https://web.telegram.org`  
   - Slack: `https://slack.com/signin`
3. **Spell checker validation**: In settings, enable/disable spell check languages and test in message inputs
4. **Tab functionality**: 
   - Switch between service tabs using Ctrl+Tab or clicking tab headers
   - Reload tabs with Ctrl+R
   - Test tab reordering by dragging tab headers
5. **Keyboard shortcuts**: Test F11 (fullscreen), Ctrl+f (find), Ctrl+[1-9] (jump to tab)
6. **Notifications**: Test that messaging notifications from services appear as system notifications

![Application Screenshot](docs/screenshots/main.png)

### Screenshots for Visual Validation
- `docs/screenshots/main.png` - Main application interface with multiple messaging tabs
- `docs/screenshots/settings-empty.png` - Empty settings dialog on first launch  
- `docs/screenshots/settings.png` - Settings with configured services and spell check options

### Browser Testing
The project includes browser tests using JSDOM and Testing Library:
- Browser test files use `.browser.test.mjs` extension
- Settings functionality can be tested at `src/settings/__tests__/settings.browser.test.mjs`

### End-to-End Testing
The project includes E2E tests to verify the complete Electron application stack:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manusa/electronim](https://github.com/manusa/electronim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
