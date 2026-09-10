---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the official **Puppet extension for Visual Studio Code** — a TypeScript VSCode extension client. It does *not* contain the Puppet language intelligence itself; that lives in the [Puppet Editor Services](https://github.com/puppetlabs/puppet-editor-services) language server (Ruby), which is downloaded and bundled at build time ("vendored"). This repo is primarily a Language Server Protocol (LSP) client plus VSCode-specific features (commands, views, debugging, status bar).

## Build & Development

The build is driven by **psake** (PowerShell), not plain npm. PowerShell (`pwsh`) and the `psake` module are required. The npm `build`/`watch` scripts shell out to `build.ps1`.

```bash
npm install                        # install node deps
npm run build                      # full build: npm install + vendor + tsc (via build.ps1 -> psakefile.ps1)
npm run compile                    # TypeScript compile only (tsc -p ./), output to ./out
npm run watch                      # build once, then tsc -watch
./build.ps1 -task clean,vendor     # re-download the language server + syntax (see Vendoring below)
```

To run/debug the extension interactively: open the repo in VSCode and press **F5** (launches an Extension Development Host). Vendored resources must exist first (`./build.ps1 -task vendor`).

## Lint, Format, Test

```bash
npm run lint            # eslint --ext .ts src
npm run fix             # eslint --fix
npm run format          # prettier --write on **/*.{ts,js,json}
npm test                # compiles, then runs integration tests in a headless VSCode (out/test/runtest.js)
npm run test:coverage   # same, with nyc/lcov coverage
```

Tests are **VSCode integration tests** (Mocha, `tdd` UI, chai + sinon), launched via `@vscode/test-electron` which downloads a real VSCode and runs the suite inside it — there is no pure-unit `mocha` runner. [src/test/runtest.ts](src/test/runtest.ts) is the entry point; [src/test/suite/index.ts](src/test/suite/index.ts) globs `**/*.test.js` from the compiled `out/test` dir. Because tests run compiled JS, **you must `npm run compile` (or `npm test` which does it via `pretest`) before test changes take effect.** There is no built-in single-test filter in the npm scripts; narrow runs by temporarily using Mocha's `.only` or commenting the glob.

Note: many tests resolve a real PDK/Puppet install path; CI installs the PDK on each OS before running (see [.github/workflows/vscode-ci.yml](.github/workflows/vscode-ci.yml)). The full suite is run across Linux/Windows/macOS.

## Vendoring (important architecture detail)

The language server and syntax grammar are **not committed** — they are fetched into `vendor/` during the build. [psakefile.ps1](psakefile.ps1) reads the `editorComponents` block in [package.json](package.json) to decide what to fetch:

- `editorServices.release` / `editorSyntax.release` — download a tagged GitHub release (default).
- `githubref` / `githubuser` / `githubrepo` — fetch from a specific repo/branch instead.
- `directory` — copy from a local checkout (useful when developing the language server alongside this extension).

The `VendorEditorServices` / `VendorEditorSyntax` tasks have preconditions that skip if `vendor/languageserver` or `syntaxes/puppet.tmLanguage` already exist, so run `clean` first to force a refresh. See [README_BUILD.md](README_BUILD.md) for all options. Cytoscape (used by the node graph) is also vendored from `node_modules`.

## Code Architecture

The extension entry point is [src/extension.ts](src/extension.ts) (`activate`/`deactivate`). Activation flow:

1. Reads VSCode workspace settings → `ISettings` ([src/settings.ts](src/settings.ts)), warns on legacy/deprecated settings.
2. Builds an **`IAggregateConfiguration`** ([src/configuration.ts](src/configuration.ts)) — this resolves the abstract settings into concrete paths: which Puppet install to use (PDK vs Agent, `auto`-detected), Ruby dirs, RUBYLIB/PATH env, SSL paths, and PDK Ruby instance discovery ([src/configuration/pdkResolver.ts](src/configuration/pdkResolver.ts), [src/configuration/pathResolver.ts](src/configuration/pathResolver.ts)). This is the single source of truth for "how do I invoke the bundled Ruby".
3. Constructs a list of **Features** and a **ConnectionHandler**.

### Features

Every discrete capability is an `IFeature` (a `vscode.Disposable`, [src/feature.ts](src/feature.ts)). They live in [src/feature/](src/feature/) and are instantiated into `extensionFeatures[]` in `activate()`, then all disposed in `deactivate()`. To add a capability, create a new `IFeature` and push it onto that array. Examples: `PDKFeature` (PDK commands), `DebuggingFeature`, `FormatDocumentFeature`, `PuppetNodeGraphFeature`, `PuppetStatusBarFeature`, the Puppetfile hover/completion features.

### ConnectionHandler (LSP client)

[src/handler.ts](src/handler.ts) defines the abstract `ConnectionHandler` wrapping a `vscode-languageclient` `LanguageClient`. Two concrete subclasses choose the transport based on `puppet.editorService.protocol`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puppetlabs/puppet-vscode](https://github.com/puppetlabs/puppet-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
