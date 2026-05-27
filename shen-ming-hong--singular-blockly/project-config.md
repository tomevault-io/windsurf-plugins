---
trigger: always_on
description: **IMPORTANT**: Always respond in **Traditional Chinese (繁體中文)** unless explicitly requested otherwise.
---

# Singular Blockly - Copilot Instructions

## Language Convention

**IMPORTANT**: Always respond in **Traditional Chinese (繁體中文)** unless explicitly requested otherwise.

## Project Overview

VS Code extension for visual Arduino/MicroPython programming using Google Blockly. Generates Arduino C++ (via PlatformIO) or MicroPython (via mpremote for CyberBrick) based on board selection. Supports 15 languages with 99% i18n coverage.

**Tech Stack**: TypeScript 5.9.3 | Blockly 12.3.1 | VS Code 1.105.0+ | MCP SDK 1.26.0 | Zod 4.1.13

**Extension dependency**: PlatformIO IDE (`platformio.platformio-ide`) is declared in `extensionDependencies`.

## Build, Test, and Lint

```powershell
npm run watch              # Webpack watch (F5 to debug in VS Code)
npm run compile            # One-off webpack build
npm run package            # Production build (--mode production)
npm run lint               # ESLint over src/
npm test                   # Full test suite via @vscode/test-cli
npm run test:coverage      # Tests with coverage report
npm run test:bail          # Stop on first failure
npm run test:integration   # Integration tests (requires Copilot)
npm run validate:i18n      # Validate all 15 locale files
npm run generate:dictionary # Rebuild MCP block-dictionary.json
```

**Run a single test file**: Tests run via `@vscode/test-cli` which launches a VS Code instance. To focus on specific tests, use `.only` in Mocha (`describe.only` / `it.only`) then run `npm test`. Test config is in `.vscode-test.mjs` with two labels: `unit` (excludes integration) and `integration`.

**Debug WebView**: F5 → Right-click Blockly panel → "Open Developer Tools"

## Architecture (Two-Context System)

The extension runs in two isolated JavaScript contexts that communicate via `postMessage`:

```
Extension Host (Node.js)              WebView (Browser)
├── src/extension.ts          ←→      ├── media/html/blocklyEdit.html
├── src/webview/                      ├── media/js/blocklyEdit.js
│   ├── webviewManager.ts             └── media/blockly/
│   └── messageHandler.ts                 ├── blocks/*.js (block definitions)
├── src/mcp/mcpServer.ts                  └── generators/{arduino,micropython}/*.js
├── src/mcp/tools/*.ts
├── src/services/
│   ├── fileService.ts         # ALL file I/O (inject FileSystem for tests)
│   ├── logging.ts             # ALL logging (never use console.log)
│   ├── settingsManager.ts     # PlatformIO config + theme + auto-backup
│   ├── arduinoUploader.ts     # PlatformIO CLI upload
│   ├── micropythonUploader.ts # mpremote upload for CyberBrick
│   ├── workspaceValidator.ts  # Workspace state integrity
│   ├── projectTypeDetector.ts # Non-Blockly project safety guard
│   └── shadowSuggestionService.ts # AI block suggestions via Copilot LM API
└── src/types/
    ├── board.ts               # BoardLanguage, UploadMethod types
    ├── arduino.ts             # getBoardLanguage(), board configs
    └── nodeDetection.ts       # Node.js detection types & MIN_NODE_VERSION
```

**Complete services inventory** (`src/services/`):
| Service | Role |
|---|---|
| `fileService.ts` | ALL file I/O — inject `FileSystem` for tests |
| `logging.ts` | ALL logging — never use `console.log` |
| `settingsManager.ts` | PlatformIO config, theme, auto-backup |
| `arduinoUploader.ts` | PlatformIO CLI upload |
| `micropythonUploader.ts` | mpremote upload for CyberBrick |
| `arduinoMonitorService.ts` | Arduino PlatformIO serial monitor |
| `serialMonitorService.ts` | CyberBrick MicroPython serial monitor (mpremote) |
| `workspaceValidator.ts` | Workspace state integrity checks |
| `projectTypeDetector.ts` | Non-Blockly project safety guard |
| `shadowSuggestionService.ts` | AI block suggestions via Copilot LM API |
| `aiModelManager.ts` | Copilot tier detection & per-tier AI config |
| `aiStatusBar.ts` | Status bar indicator for AI suggestion state |
| `localeService.ts` | Runtime locale/UI messages loader |
| `nodeDetectionService.ts` | Node.js availability & version validation |
| `diagnosticService.ts` | VS Code Diagnostic collection management |

**Data Flow**: WebView `saveWorkspace` → `messageHandler.ts` → `FileService` → `blockly/main.json`

**Key constraint**: Extension Host code (TypeScript in `src/`) cannot import WebView code (`media/`), and vice versa. They only communicate via `postMessage`.

## Critical Patterns

### Extension ↔ WebView Communication

```typescript
// Extension → WebView (in messageHandler.ts)
panel.webview.postMessage({ command: 'loadWorkspace', state: {...}, board: 'esp32' });

// WebView → Extension (in blocklyEdit.js)
vscode.postMessage({ command: 'saveWorkspace', state: {...}, board: currentBoard });

// Add handlers in messageHandler.ts switch-case:
case 'newCommand': await this.handleNewCommand(message); break;
```

### Dual Code Generators (Board-Aware)

Each block needs both an Arduino and MicroPython generator. They use different patterns:

**Arduino** (`media/blockly/generators/arduino/*.js`):

```javascript
arduinoGenerator.forBlock['servo_setup'] = function (block) {
	const currentBoard = window.getCurrentBoard(); // 'uno' | 'esp32' | 'mega'
	if (currentBoard === 'esp32') {
		arduinoGenerator.lib_deps_.push('madhephaestus/ESP32Servo@^3.0.6');
	} else {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shen-Ming-Hong/singular-blockly](https://github.com/Shen-Ming-Hong/singular-blockly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
