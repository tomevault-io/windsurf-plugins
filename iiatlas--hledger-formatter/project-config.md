---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An npm workspaces monorepo containing a formatter for hledger journal files, published as both a VS Code extension and a standalone CLI tool. Supports `.journal`, `.hledger`, and `.ledger` file extensions.

### Monorepo Structure

```
packages/
  formatter/   @hledger-fmt/formatter  — pure formatting logic (private, never published)
  cli/         hledger-fmt             — CLI tool (published to npm)
  vscode/      hledger-formatter       — VS Code extension (published to Marketplace)
```

- **formatter** has zero dependencies and no vscode imports. All other packages import from `@hledger-fmt/formatter`.
- **cli** bundles formatter + commander into a single file via esbuild.
- **vscode** bundles formatter via esbuild, externalizes the `vscode` module.

## Development Commands

From the repository root:

- `npm run build` — Build all packages (formatter → cli → vscode, order matters)
- `npm run test` — Run vitest in packages/formatter (62 tests, ~120ms)
- `npm run lint` — ESLint across all packages
- `npm run watch` — Watch mode for the VS Code extension
- `npm run clean` — Remove out/ and dist/ from all packages

### CLI usage

- `node packages/cli/bin/hledger-fmt.js format [file]` — Format a journal file (or stdin)
- `node packages/cli/bin/hledger-fmt.js sort [file]` — Sort a journal file by date

### Packaging

- `cd packages/vscode && npx vsce package --no-dependencies` — Build .vsix

## Architecture

### packages/formatter (`@hledger-fmt/formatter`)

Pure formatting logic with zero external dependencies. Built with `tsc` (composite + declaration) so other packages can consume types.

**Key files:**
- `src/formatter.ts` — All formatting, sorting, comment toggling, amount parsing logic
- `src/index.ts` — Barrel export (`export * from './formatter'`)
- `test/formatter.test.ts` — 62 tests using vitest
- `test/test_journals/` — Input/output journal pairs for test fixtures

**Exported functions:**
- `formatHledgerJournal()` — Main formatting function
- `formatTransaction()` — Formats individual transactions
- `formatTransactionHeader()` — Normalizes transaction header spacing
- `toggleCommentLines()` — Toggle comments on selected lines
- `sortHledgerJournal()` — Sorts journal entries by date
- `parseAmount()` — Parses amount strings, extracts value/currency
- `formatAmountValue()` — Formats numeric values with currency symbols
- `calculateBalancingAmount()` — Calculates balancing amounts for transactions

### packages/cli (`hledger-fmt`)

CLI entry point using commander. esbuild bundles everything (formatter + commander) into `dist/cli.js`. The shebang wrapper at `bin/hledger-fmt.js` requires the bundle.

### packages/vscode (`hledger-formatter`)

VS Code extension glue. esbuild bundles formatter into `dist/extension.js` with `vscode` as external.

**Key files:**
- `src/extension.ts` — Extension activation, command registration, providers
- `syntaxes/hledger.tmLanguage.json` — TextMate grammar for syntax highlighting
- `language-configuration.json` — Bracket/comment configuration
- `images/` — Icon and demo GIFs

**Commands:**
- `hledger-formatter.formatDocument` — Manual format
- `hledger-formatter.toggleComment` (Cmd+/) — Toggle comments
- `hledger-formatter.sortEntries` (Shift+Cmd+S) — Sort entries by date
- `hledger-formatter.newFile` (Cmd+N in .journal file) — Create new monthly journal

**Providers:**
- Format-on-save handler (with optional sort-on-save)
- Document formatting provider (integrates with VS Code's Format Document)
- Range formatting provider
- Inline completion provider (balancing amount ghost text)
- Account autocomplete provider

### Build Pipeline

Build order matters: formatter must build first (produces declarations), then cli + vscode consume it.

- **formatter**: `tsc` → `out/` (JS + declarations)
- **cli**: `esbuild` → `dist/cli.js` (single bundled file)
- **vscode**: `esbuild` → `dist/extension.js` (single bundled file, vscode external)

Workspace resolution: esbuild configs use `nodePaths: [path.resolve(__dirname, '../../node_modules')]` to resolve `@hledger-fmt/formatter` through the workspace symlink in root `node_modules/`.

### Formatting Logic

The formatter aligns all amounts to a configurable column position (default: 42 characters from left):
- Uses exactly 2 spaces for posting line indentation
- Handles negative amounts by converting `$-X.XX` to `-$X.XX` format
- Preserves comments and transaction structure
- Normalizes transaction header spacing
- Column position is configurable via `hledger-formatter.amountColumnPosition` setting

### Comment Toggle Logic

The comment toggle feature (Cmd+/) uses smart block behavior and preserves indentation:

**Supported Comment Formats:**
- `;` (semicolon) — Most common, recommended for temporary commenting
- `#` (hash/pound) — Recommended for top-level notes
- `*` (asterisk) — Useful for Emacs org-mode users
- `comment`/`end comment` — Multi-line comment blocks (preserved during formatting)

**Smart Block Behavior:**
- If any uncommented lines exist in selection → comments ALL lines
- If all lines are commented → uncomments ALL lines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iiAtlas/hledger-formatter](https://github.com/iiAtlas/hledger-formatter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
