---
trigger: always_on
description: Cloak is a **VS Code extension** that hides and shows secrets in environment files (`.env`). It manipulates TextMateRules to set the foreground color to transparent, so secrets are invisible while presenting, streaming, or recording.
---

# Cloak — Agent Guide

Cloak is a **VS Code extension** that hides and shows secrets in environment files (`.env`). It manipulates TextMateRules to set the foreground color to transparent, so secrets are invisible while presenting, streaming, or recording.

## Repository Structure

```
vscode-cloak/
├── src/
│   ├── extension.ts              # Activation entry point, command registration
│   ├── commands.ts               # Command handlers (hide, show, toggle, restoreDefaults)
│   ├── statusbar.ts              # Status bar item (eye icon, toggle command)
│   ├── logging.ts                # Output channel logger
│   ├── configuration/
│   │   ├── index.ts              # Re-exports
│   │   ├── read-configuration.ts # Read VS Code settings and textMateRules
│   │   └── update-configuration.ts # Write VS Code settings
│   ├── models/
│   │   ├── index.ts              # Re-exports + getExtension() helper
│   │   ├── constants.ts          # extensionShortName, extensionId, utility fns
│   │   ├── enums.ts              # Commands, Settings, Sections, TextMateRulesNames, TextMateScopeDefaults
│   │   └── interfaces.ts         # ICommand, IConfiguration, ISettings
│   └── test/
│       ├── runTest.ts            # Test runner entry (vscode-test)
│       ├── coverage.ts           # Istanbul coverage instrumentation
│       └── suite/
│           ├── index.ts          # Mocha config (TDD UI, xunit reporter)
│           ├── basic.test.ts     # Extension activation, command/setting registration tests
│           ├── extension.test.ts # Placeholder test suite
│           └── lib/
│               ├── constants.ts          # Test helpers (executeCommand)
│               └── setup-teardown-test-suite.ts # Save/restore settings between tests
├── resources/                    # Extension icon and screenshots
├── dist/                         # Webpack output (not committed)
├── .vscode/                      # Dev environment (launch configs, tasks)
├── webpack.config.js             # Bundles src/extension.ts → dist/extension.js
├── tsconfig.json                 # TypeScript config (commonjs, es6, strict)
├── .eslintrc                     # ESLint + TypeScript parser
├── .prettierrc.js                # Prettier (single quotes, trailing commas, 100 width)
├── package.json                  # Extension manifest — commands, settings, menus, keybindings
├── CHANGELOG.md                  # Version history
├── README.md                     # User-facing docs
└── LICENSE.md                    # MIT
```

## Tech Stack

- **Language:** TypeScript
- **Runtime:** VS Code Extension Host (Node.js)
- **Bundler:** webpack (ts-loader)
- **Test runner:** Mocha (TDD UI) via `vscode-test`
- **Linter:** ESLint with `@typescript-eslint`
- **Formatter:** Prettier
- **Coverage:** Istanbul
- **Extension dependency:** `mikestead.dotenv` (provides TextMate scopes for `.env` files)

## Build & Run

```bash
# Install dependencies
npm install

# Compile TypeScript + webpack bundle
npm run webpack

# Production bundle (used by vsce)
npm run vscode:prepublish

# Watch mode for development
npm run watch

# Package the extension (.vsix)
npm run package

# Publish to VS Code Marketplace
npm run publish
```

To debug locally, press `F5` in VS Code — this launches an Extension Development Host with the extension loaded (see `.vscode/launch.json`).

## Testing

```bash
# Full test: compile TypeScript → webpack → run Mocha in VS Code host
npm test

# Run tests only (skip compile, assumes already built)
npm run just-test
```

Tests run inside a VS Code instance via `vscode-test`. The test runner opens a `testworkspace` folder with extensions disabled. Tests use Mocha's TDD interface (`suite`, `test`, `suiteSetup`, `suiteTeardown`).

Key test file: `src/test/suite/basic.test.ts` — verifies extension activation, command registration against `package.json`, and setting registration.

## Key Patterns and Conventions

- **`package.json` is the product spec** — all commands, settings, menus, and keybindings are declared in `contributes`. This is the source of truth for what the extension exposes to users.
- **Command registration chain:** Commands are declared in `package.json` → enum IDs live in `src/models/enums.ts` (`Commands` enum) → handlers are in `src/commands.ts` → registration happens in `src/extension.ts` (`registerCommands()`).
- **Configuration pattern:** Settings are declared in `package.json` `contributes.configuration` → read via `src/configuration/read-configuration.ts` → written via `src/configuration/update-configuration.ts`. Setting keys are in the `Settings` enum.
- **TextMateRules mechanism:** Cloak works by injecting/removing TextMateRules into `editor.tokenColorCustomizations`. It sets foreground to `#19354900` (alpha 0 = transparent) to hide values. The scope defaults are in the `TextMateScopeDefaults` enum.
- **Re-export pattern:** Both `src/models/index.ts` and `src/configuration/index.ts` re-export all members from their submodules.
- **Status bar:** A left-aligned status bar item with an eye icon, bound to `toggleSecrets`.

## Adding a New Command


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpapa/vscode-cloak](https://github.com/johnpapa/vscode-cloak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
