---
trigger: always_on
description: This file provides context for AI coding assistants (Claude, GPT, Copilot, Gemini, etc.)
---

# AGENTS.md - AI Assistant Guide for DemoMonkey

This file provides context for AI coding assistants (Claude, GPT, Copilot, Gemini, etc.)
working on the DemoMonkey codebase.

## Project Overview

DemoMonkey is a **Chrome/Firefox browser extension** (Manifest V3) that lets users customize
web application UIs for prospect-specific demos. Users write INI-based configuration files
with text/visual replacement rules, and the extension applies them to live web pages in
real time.

- **License:** Apache 2.0
- **Language:** JavaScript (ES6+) with JSX
- **UI Framework:** React 19 + Redux + Material-UI (MUI)
- **Build Tool:** Webpack 5 + Babel 7
- **Tests:** Mocha/Chai (unit), Playwright (E2E)
- **Linting:** ESLint 9 (flat config) + Prettier

## Quick Reference Commands

```bash
npm install          # Install dependencies
npm start            # Dev build with watch mode (output: build/)
npm run build        # One-off dev build
npm test             # Run unit tests (mocha)
npm run test:e2e     # Run E2E tests (playwright)
npm run test:all     # Run unit + E2E tests
npm run lint         # Check linting errors
npm run lint:fix     # Auto-fix linting errors
npm run format       # Check formatting (prettier)
npm run format:fix   # Auto-fix formatting
npm run release      # Production build + ZIP for Chrome Web Store
```

## Project Structure

```text
src/
  app.js              # UI entry point (popup + options page)
  monkey.js           # Content script (injected into web pages, applies configs)
  background.js       # Service worker (extension lifecycle, messaging)
  inline.js           # Injected script for AJAX request interception
  backup.js           # Configuration backup utility

  models/             # Core business logic
    Monkey.js         # Main replacement engine (DOM walking, undo tracking)
    Configuration.js  # Parses INI configs, manages variables/options
    CommandBuilder.js # Builds Command objects from config patterns
    Repository.js     # Handles config imports and dependencies
    Settings.js       # Global extension settings
    Variable.js       # Template variable substitution
    MatchRule.js      # URL include/exclude matching
    ...

  commands/           # 40+ replacement/manipulation commands
    Command.js        # Base class for all commands
    CommandBuilder.js # Registry + factory for command instances
    SearchAndReplace.js, Hide.js, Style.js, ReplaceImage.js, ...
    appdynamics/      # AppDynamics-specific commands

  components/         # React UI
    options/          # Dashboard/options page (editor, navigation, settings)
    popup/            # Browser action popup
    shared/           # Reusable UI components

  reducers/           # Redux state (configurations, settings, log)
  helpers/            # Utility functions
  styles/             # LESS stylesheets
  pages/              # HTML templates

test/
  commands/           # Command unit tests
  models/             # Model unit tests
  helpers/            # Helper unit tests
  playwright/         # E2E browser tests
```

## Architecture

### Data Flow

```text
INI config text
  -> Ini.js (parse to key-value pairs)
  -> Configuration.js (extract options, variables, imports)
  -> CommandBuilder.js (create Command instances)
  -> Monkey.js (walk DOM, apply commands at intervals)
  -> UndoElement tracking (for reverting changes)
```

### Extension Components

| Entry Point         | Role                                               |
| ------------------- | -------------------------------------------------- |
| `src/app.js`        | Renders React UI for popup and options page        |
| `src/monkey.js`     | Content script: applies configurations to page DOM |
| `src/background.js` | Service worker: manages tabs, messaging, requests  |
| `src/inline.js`     | Injected into pages for AJAX/XHR interception      |

### State Management

Redux with **webext-redux** to sync state between the background service worker
and content scripts via message passing. Key reducers: `configurations`, `settings`,
`log`, `connectionState`.

### INI Configuration Format

```ini
; Options
@include[] = /regex/         ; URL include rules
@exclude[] = /regex/         ; URL exclude rules
@namespace[] = appdynamics   ; Import command namespaces

; Simple replacement
search_text = replacement_text

; Commands (prefix with !)
!hide(selector)
!style(selector) = color: red
!/regex/i = replacement

; Variables (prefix with $)
$customer = Acme Corp//Customer name

; Imports (prefix with +)
+SharedConfig
```

## Code Conventions

### File Headers (Required)

Every `.js` file in `src/` and `test/` must start with the Apache 2.0 license header:

```javascript
/**
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [svrnm/DemoMonkey](https://github.com/svrnm/DemoMonkey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
