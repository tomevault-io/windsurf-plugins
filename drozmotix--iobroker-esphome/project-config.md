---
trigger: always_on
description: **Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions
---

# ioBroker Adapter Development with GitHub Copilot

**Version:** 0.4.2
**Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions

This file contains instructions and best practices for GitHub Copilot when working on ioBroker adapter development.

## Project Context

You are working on an ioBroker adapter. ioBroker is an integration platform for the Internet of Things, focused on building smart home and industrial IoT solutions. Adapters are plugins that connect ioBroker to external systems, devices, or services.

### ioBroker.esphome Adapter

ioBroker.esphome is a Node.js adapter for ioBroker that integrates ESP8266/ESP32 devices managed by ESPHome. The adapter communicates with devices via ESPHome's native API and optionally provides an integrated ESPHome Dashboard using Python.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

#### Working Effectively
- Bootstrap, build, and test the repository:
  - `npm ci` -- installs dependencies. Takes 40 seconds. NEVER CANCEL. Set timeout to 90+ seconds.
  - `npm run lint` -- ESLint check. Takes <5 seconds.
  - `npm run check` -- TypeScript check. Takes <5 seconds. May show type errors that are non-blocking.
  - `npm test` -- runs all tests. Takes 30 seconds. NEVER CANCEL. Set timeout to 60+ minutes.
- Development server for local testing:
  - `npm run dev-server setup` -- one-time setup. Takes 90 seconds. NEVER CANCEL. Set timeout to 180+ seconds.
  - `npm run dev-server run default` -- starts full ioBroker instance with admin UI on http://127.0.0.1:8081/
  - `npm run startDev` -- alternative development mode (requires setup first)
- Translation management:
  - `npm run translate` -- updates translations using Gulp. Takes <1 second.

#### Validation
- Always run the full test suite after making changes: `npm test`
- ALWAYS run `npm run lint` before you are done or the CI (.github/workflows/test-and-release.yml) will fail.
- **ESLint warnings are treated as errors in CI** (`--max-warnings 0`). Always run `./node_modules/.bin/eslint --max-warnings 0 .` to catch warnings that would be ignored by `npm run lint` locally but fail in CI. Fix ALL warnings — not just errors — before finishing a PR.
- The TypeScript check (`npm run check`) may show errors but they are non-blocking and do not prevent the adapter from functioning.
- Always test actual ESPHome device integration scenarios when making changes to device communication or API handling.
- Test the admin UI configuration when making changes to adapter settings or device management.
- For functional testing, use `npm run dev-server run default` to start a full ioBroker instance with admin UI at http://127.0.0.1:8081/
- When testing device integration, ensure ESPHome API is enabled in device YAML with either encryption key or password authentication.
- Test translation changes with `npm run translate` to ensure all language files are properly updated.

#### Common Tasks
The following are outputs from frequently run commands. Reference them instead of viewing, searching, or running bash commands to save time.

##### Node.js and npm versions
node --version: v20.19.5
npm --version: 10.8.2

##### Key dependencies
- @2colors/esphome-native-api: ESPHome device communication
- @iobroker/adapter-core: ioBroker adapter framework
- autopy: Python virtual environment for ESPHome Dashboard
- node-fetch: HTTP requests for API communication

##### Repository structure
```
ls -la [repo-root]:
.eslintrc.json          -- ESLint configuration
.github/                -- GitHub workflows and templates
.gitignore
admin/                  -- Admin UI files (HTML, CSS, translations)
io-package.json        -- ioBroker adapter configuration
lib/                   -- Helper modules
  helpers.js           -- Device client management (113 lines)
  stateAttr.js         -- State attribute definitions (47 lines) 
  tools.js             -- Utility functions and translation (99 lines)
main.js                -- Main adapter code (1737 lines)
package.json           -- Node.js project configuration
test/                  -- Test files
  integration.js       -- Adapter startup tests
  package.js           -- Package validation tests
  unit.js              -- Unit tests (minimal)
tsconfig.json          -- TypeScript configuration
```

##### Test structure
- **test:package** -- validates package.json and io-package.json structure (~instant)
- **test:unit** -- minimal unit tests (deprecated, ~instant)
- **test:integration** -- starts adapter in test environment (~28 seconds)
- **test:js** -- runs basic JavaScript tests (~instant)

##### Key scripts (package.json)
- `npm run lint` -- ESLint code quality check
- `npm run check` -- TypeScript type checking  
- `npm test` -- run full test suite (package + unit + integration + js)
- `npm run startDev` -- development server (requires setup)
- `npm run dev-server setup` -- one-time development environment setup
- `npm run translate` -- update translations via Gulp

##### Development workflow
1. Make code changes to main.js or lib/ files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrozmotiX/ioBroker.esphome](https://github.com/DrozmotiX/ioBroker.esphome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
