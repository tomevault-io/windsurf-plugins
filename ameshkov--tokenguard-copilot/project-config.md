---
trigger: always_on
description: handles, and VS Code API registrations — MUST be explicitly disposed on
---

# AGENTS.md

VS Code extension that provides third-party OpenAI-compatible language models to
VS Code Copilot Chat via the `languageModelChatProvider` API. Registers as a
chat model provider so any OpenAI-compatible endpoint can be used as a backend
for Copilot Chat.

## Table of Contents

- [Project Overview](#project-overview)
- [Technical Context](#technical-context)
- [Project Structure](#project-structure)
- [Build and Test Commands](#build-and-test-commands)
- [Contribution Instructions](#contribution-instructions)
- [Code Guidelines](#code-guidelines)
    - [Architecture](#architecture)
    - [Code Quality](#code-quality)
    - [Testing](#testing)
    - [Dependency Management](#dependency-management)
    - [Configuration & Documentation](#configuration--documentation)
    - [Markdown Formatting](#markdown-formatting)

## Project Overview

A VS Code extension that bridges third-party OpenAI-compatible language model
endpoints into VS Code Copilot Chat.
It uses the proposed `languageModelChatProvider` API to register external models
so they appear alongside built-in Copilot models.

What you get out of the box:

- **Chat model provider** — registers OpenAI-compatible models as VS Code
  language model chat providers.
- **Copilot Chat integration** — models appear in the Copilot Chat model picker
  and can be used by any chat participant.
- **Code quality** — ESLint (flat config), Prettier, Markdownlint, Husky
  pre-commit hooks.

## Technical Context

| Field | Value |
| --- | --- |
| Language | TypeScript 5.9, ES2022 target, strict mode |
| Runtime | VS Code Extension Host (Node.js) |
| Package Manager | pnpm |
| VS Code API | `^1.120.0` (proposed `chatProvider` API) |
| Linting | ESLint 9.x + typescript-eslint |
| Formatting | Prettier 3.x, Markdownlint (markdownlint-cli2) |
| Testing | Vitest 4.x (unit), @vscode/test-cli (E2E) |
| Project Type | VS Code extension |

## Project Structure

The extension has two distinct runtime targets that share types:

1. **Extension host** — Node.js code running in the VS Code extension host
   (providers, models, database, chat completion).
2. **Webview UI** — browser code (React) running inside a VS Code webview panel
   (settings, model configuration).

A pnpm monorepo lets us share TypeScript types between host and webview, run
separate bundlers per target, and produce a single `.vsix`. The root
`package.json` doubles as the VS Code extension manifest (`main`, `contributes`,
`publisher`).

### Packages

- **`packages/shared`** (`@tokenguard/shared`) — shared TypeScript types and
  message protocol definitions used by both the extension host and the webview.
- **`packages/extension`** (`@tokenguard/extension`) — extension host code:
  activation, providers, services, database layer.
  Bundled with esbuild into `out/extension.js` (CJS, Node.js).
- **`packages/webview-ui`** (`@tokenguard/webview-ui`) — React settings UI.
  Bundled with esbuild into `out/webview/` (IIFE, browser).
- **`packages/webview-playground`** (`@tokenguard/webview-playground`) — Vite
  dev server with `@vscode-elements/webview-playground` toolbar and mock VS Code
  API for developing the settings page in a browser.

### Directory Layout

```text
tokenguard-copilot/
├── pnpm-workspace.yaml          # Workspace: packages/*
├── package.json                 # Extension manifest + root scripts
├── tsconfig.json                # Base TS config (shared settings)
├── eslint.config.mjs            # ESLint flat config
├── knip.config.ts               # Knip unused-export config
├── .vscode-test.mjs             # E2E test runner config
├── assets/                      # Static assets shipped with extension
│   ├── model-defaults.json      # Bundled model defaults database
│   └── webview/
│       └── settings.html        # Webview HTML shell template
├── test-e2e/                    # E2E tests (separate from packages)
│   ├── tsconfig.json
│   ├── helpers.ts               # Shared E2E test utilities
│   ├── mock-openai-server.ts    # Mock OpenAI-compatible HTTP server
│   ├── extension.test.ts        # Extension activation tests
│   ├── commands.test.ts         # Command registration tests
│   ├── settings-panel.test.ts   # Webview panel tests
│   ├── debug-logging.test.ts    # Debug logging command tests
│   ├── tree-view.test.ts        # Tree view registration tests
│   ├── database.test.ts         # Database lifecycle tests
│   └── chat-completion.test.ts  # Provider + model + chat E2E tests
├── packages/
│   ├── shared/                  # Shared types & protocol
│   │   ├── package.json         # @tokenguard/shared
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── index.ts         # Barrel exports
│   │       └── messages.ts      # Host ↔ webview message protocol
│   ├── extension/               # Extension host (VS Code extension)
│   │   ├── package.json         # @tokenguard/extension
│   │   ├── tsconfig.json
│   │   ├── vitest.config.mts    # Unit test config
│   │   ├── esbuild.config.mts   # Node.js bundle config
│   │   ├── drizzle.config.ts    # Drizzle Kit config
│   │   └── src/
│   │       ├── extension.ts     # activate() / deactivate()
│   │       ├── context.ts       # ExtensionContext (DI container)
│   │       ├── commands/        # Command handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ameshkov/tokenguard-copilot](https://github.com/ameshkov/tokenguard-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
