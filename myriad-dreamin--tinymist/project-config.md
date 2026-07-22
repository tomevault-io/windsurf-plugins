---
trigger: always_on
description: This is the VS Code extension for the tinymist language service, providing language features and additional tools like preview rendering for Typst projects within the VS Code editor.
---

# AGENTS.md

This is the VS Code extension for the tinymist language service, providing language features and additional tools like preview rendering for Typst projects within the VS Code editor.

## Quick Start

1. Read `editors/vscode/package.json` and the nearest files under `src/` before editing.
2. For behavior changes, also inspect the Rust side that backs the extension, especially `crates/tinymist`, `crates/tinymist-cli`, `crates/tinymist-project`, and `crates/tinymist-query`.
3. If the task changes behavior in a non-trivial way, check OpenSpec artifacts in `openspec/specs/` and `openspec/changes/` from the repo root.
4. Before finishing, run the smallest relevant extension validation and note what you verified.

## Source Of Truth

- Edit source files under `editors/vscode/src/` for extension logic.
- Edit `editors/vscode/package.json` for extension manifest, commands, contributions, and configuration that ships in the main manifest.
- Edit `editors/vscode/package.other.json` for additional configuration definitions that are consumed by the shared docs pipeline.
- Edit `locales/tinymist-vscode.toml` and `locales/tinymist-vscode-rt.toml` for localization source text. Before that, from the repository root, run `yarn build:l10n` to extract text to translate for you.
- Edit `docs/tinymist/frontend/vscode.typ` and `docs/tinymist/config/vscode.typ` for generated VS Code docs.
- Edit `syntaxes/textmate/` for grammar changes, not the generated copies bundled into the extension.

## Do Not Hand-Edit Generated Outputs

Besides the syntax files, all other generated assets are outputs, not sources. Do not edit them directly.

- If syntax files under `editors/vscode/out/*.tmLanguage.json` need to change, update `syntaxes/textmate/` and rebuild.

## Repo Map

- `src/extension.ts`: main desktop/system extension entrypoint.
- `src/extension.web.ts`: web extension entrypoint with a reduced feature set.
- `src/extension.shared.ts`: shared activation and shutdown flow.
- `src/lsp.ts`: shared language-client state and common LSP wiring.
- `src/lsp.system.ts`: system binary launch path for the bundled or configured `tinymist` executable.
- `src/lsp.web.ts`: browser/web worker client path.
- `src/features/`: user-facing VS Code features such as preview, export, tasks, testing, labels, packages, drag-and-drop, and dev-kit integrations.
- `src/tools/`: webview and tool-panel integrations such as symbol view, docs view, templates, fonts, summaries, and profiling.
- `src/test/e2e/`: VS Code integration tests and workspace-driven scenarios.
- `e2e-workspaces/`: sample projects used by VS Code integration tests.

## Common Change Patterns

### Extension-only UI or command work

- Start in `src/extension.ts`, `src/extension.shared.ts`, `src/features/`, or `src/tools/`.
- If you add commands, menus, settings, views, or key contributions, update `package.json` too.
- If the user-facing text changes, update localization sources and regenerate localized JSON packs.

### Config work

- Check both the extension manifest side and the Rust server side.
- Server-backed settings often also require updates in `crates/tinymist/src/config.rs`.
- If configuration docs should change, update the Typst docs source and regenerate `Configuration.md`.

Example: a setting like `tinymist.serverPath` or `tinymist.fontPaths` spans `editors/vscode/package.json`, `src/config.ts`, `../../crates/tinymist/src/config.rs`, and `../../docs/tinymist/config/vscode.typ`.

### LSP or protocol work

- Extension code in `src/lsp*.ts` is often only one half of the change.
- Inspect the Rust request handler, config parser, or command implementation before assuming the fix is frontend-only.

Example: preview and export commands are surfaced through `src/lsp.ts`, but the server commands are registered on the Rust side in `../../crates/tinymist/src/server.rs` and implemented under files like `../../crates/tinymist/src/tool/preview.rs` or `../../crates/tinymist/src/task/export.rs`.

### Preview, export, or testing work

- These features often cross the TypeScript frontend and Rust backend boundary.
- Validate the specific workflow you touched, not just TypeScript unit tests.

Example: preview work commonly touches `src/features/preview.ts`, `src/lsp.ts`, `../../crates/tinymist/src/tool/preview.rs`, and an e2e file such as `src/test/e2e/simple-docs.test.ts`.

### Web vs system work

- Desktop/system mode uses `src/extension.ts` plus `src/lsp.system.ts`.
- Web mode uses `src/extension.web.ts` and supports a smaller feature set.
- If you touch activation, capability flags, or shared client code, consider whether both system and web builds still make sense.

Example: when changing startup or feature flags, compare `src/extension.ts` and `src/extension.web.ts` side by side so you do not accidentally assume browser builds support desktop-only features like the bundled binary.

## Testing

- Add or update tests for behavior changes whenever the affected area already has an established testing pattern.
- Use Vitest in `src/*.test.ts` for pure TypeScript logic that does not need a running VS Code instance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Myriad-Dreamin/tinymist](https://github.com/Myriad-Dreamin/tinymist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
