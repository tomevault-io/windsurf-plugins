---
trigger: always_on
description: - **Website**: https://mmt.dev (NOT multimeter.dev). Test server: https://test.mmt.dev
---

# Copilot Instructions for `multimeter` (mmt)

## Project structure & architecture
- **Website**: https://mmt.dev (NOT multimeter.dev). Test server: https://test.mmt.dev
- **Monorepo layout**:
  - `core/`: pure TypeScript library with all parsing, execution and network logic (no VS Code, no `fs` – use dependency injection).
  - `mmtview/`: React + VS Code webview UI for editing/running `.mmt` files.
  - `mmtcli/`: CLI app; binary is `testlight`, used for CI and local runs.
  - Root `src/`: VS Code extension host code (activation, editor provider, assistant, network bridge).
  - `docs/`: user-facing documentation (API, test, env, suite, CLI, convertor, etc.).
  - `AI/`: internal AI-assisted development artifacts. Not shipped to users.
    - `sdd/`: Software Design Documents (SDDs) — architectural decisions, competitive strategy, feature designs.
    - `skill/`: reusable skill/procedure documents (e.g. release & deploy workflows).
    - `doc/`: handover and context documents for AI agent sessions.
- **Single source of truth** for running `.mmt` files is `core/src/runner.ts`:
  - Use `runner.runFile({ rawFile, filePath, inputs, envvar, fileLoader, jsRunner, logger })`.
  - Do **not** reimplement parsing or execution pipelines in the extension or CLI – always go through `runner`.

## Core library (`core/`) patterns
- Keep `core` platform-neutral:
  - No imports from `vscode`, `fs`, `path`, browser APIs, or Node globals that assume a specific runtime.
  - All file access, code execution, logging, and network plumbing come in via injected functions (`fileLoader`, `jsRunner`, `logger`, etc.).
- Key modules to know:
  - `runner.ts`: orchestrates `.mmt` execution, builds API/test runners, and formats logs/docs.
  - `JSer.ts`, `testParsePack.ts`, `apiParsePack.ts`: turn YAML `.mmt` into executable JS flows.
  - `networkCore.ts`, `network.ts`, `NetworkData.ts`: HTTP/WebSocket client, message routing, and shared network config types.
  - `outputExtractor.ts` + `pathAtPosition.test.ts`: JSON/XML/xpath/jsonpath/regex extraction and “path at cursor” helpers.
  - `variableReplacer.ts`: **single source of truth** for all token-matching regex and replacement logic. All regex patterns and token accessor handling for `e:`, `i:`, `r:`, `c:` tokens (including forms like `[0]`, `[0:3]`, and `.field`) live here. Other modules (`JSerHelper`, `JSerTestFlow`, `JSerAPI`, etc.) import helpers from `variableReplacer` — they must NOT define their own token-matching regexes.
- When extending behavior:
  - First update the relevant data model types in `core/src/*Data.ts` (e.g. `APIData`, `TestData`, `NetworkConfig`).
  - Add/adjust unit tests in `core/src/*.test.ts` that cover the new pure logic.

## VS Code extension + webview
- Extension host (`src/`):
  - `extension.ts`: entrypoint; registers the `.mmt` custom editor, side panels (history, mock server, environment, certificates), and chat participants from `src/assistant.ts`.
  - `mmtEditorProvider.ts`: glue between webview messages and `runner.runFile`. It:
    - Receives `command` messages like `runCurrentDocument`, `runCurlCommand`, etc.
    - Calls `runner.runFile({ rawFile: document.getText(), filePath: document.uri.fsPath, inputs, envvar, fileLoader, jsRunner, logger })`.
  - `vscodeNetwork.ts`: adapts VS Code configuration and environment to `NetworkConfig` and bridges webview/network messages into `core`.
- Webview React app (`mmtview/src/`):
  - Uses `window.vscode.postMessage` via helpers in `vsAPI.ts` instead of importing `core` directly.
  - `text/YamlEditorPanel.tsx` drives run glyphs for `.mmt` files; it sends structured `inputs` shapes (`type: 'defaults' | 'manual' | 'exampleId' | 'exampleIndex'`) back to the extension.
  - Keep UI logic (layout, focus, interactions) here; keep parsing/execution in `core`.

## CLI (`mmtcli/`) workflow
- Entrypoint: `mmtcli/src/cli.ts` wraps `core` and exposes the `testlight` binary.
- Typical usage (see `mmtcli/README.md` and `docs/testlight.md`):
  - `npx testlight run path/to/test.mmt`
  - Pass env via `--env-file`, `--preset`, and `-e KEY=VALUE` flags; types are coerced by `coerceCliValue`/`parsePairs` in `cli.ts` (unquoted numbers/bools → numbers/bools, quoted → strings).
- If you add new CLI flags, wire them through to `runner.runFile` rather than duplicating parsing/execution.

## `.mmt` data model and docs
- `.mmt` is YAML with `type` driving behavior, parsed by `JSer.fileType`:
  - `type: api` → HTTP/WebSocket API definitions (see `docs/api-mmt.md`).
  - `type: test` → executable test flows (`call`, `assert`, `check`, etc.; see `docs/test-mmt.md`).
  - `type: env` → environment and variable files (see `docs/environment-mmt.md`).
- Converters and docs:
  - `core/src/openapiConvertor.ts`, `postmanConvertor.ts`: turn OpenAPI/Postman into `.mmt` API/test files.
  - `core/src/docHtml.ts`, `docMarkdown.ts`, `docParsePack.ts` and `res/doc-template.html`: generate HTML/Markdown API docs from `.mmt`.

## Network, logging, and errors
- All HTTP/WS traffic flows through `core/src/networkCore.ts` + `core/src/network.ts` using `NetworkConfig` from `NetworkData.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mshobeyri/multimeter](https://github.com/mshobeyri/multimeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
