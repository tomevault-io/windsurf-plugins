---
trigger: always_on
description: Adds a kanban layout to [Obsidian Bases](https://help.obsidian.md/bases) so you can display notes as cards organized in columns.
---

## Kanban view for Obsidian Bases

Adds a kanban layout to [Obsidian Bases](https://help.obsidian.md/bases) so you can display notes as cards organized in columns.

-   Display notes as cards grouped by a property value
-   Configure which property defines the columns (e.g., status, priority)
-   Customize card title and description properties
-   Click cards to open the corresponding note

## Project overview

-   Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
-   Entry point: `main.ts` compiled to `main.js` and loaded by Obsidian.
-   Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.

## Environment & tooling

-   Node.js: use current LTS (Node 18+ recommended).
-   **Package manager: npm** (required for this sample - `package.json` defines npm scripts and dependencies).
-   **Bundler: esbuild** (required for this sample - `esbuild.config.mjs` and build scripts depend on it). Alternative bundlers like Rollup or webpack are acceptable for other projects if they bundle all external dependencies into `main.js`.
-   Types: `obsidian` type definitions.

**Note**: This sample project has specific technical dependencies on npm and esbuild. If you're creating a plugin from scratch, you can choose different tools, but you'll need to replace the build configuration accordingly.

### Install

```bash
npm install
```

### Dev (watch)

```bash
npm run dev
```

### Production build

```bash
npm run build
```

## Linting

-   To use eslint install eslint from terminal: `npm install -g eslint`
-   To use eslint to analyze this project use this command: `eslint main.ts`
-   eslint will then create a report with suggestions for code improvement by file and line number.
-   If your source code is in a folder, such as `src`, you can use eslint with this command to analyze all files in that folder: `eslint ./src/`

## File & folder conventions

-   **Organize code into multiple files**: Split functionality across separate modules rather than putting everything in `main.ts`.
-   Source lives in `src/`. Keep `main.ts` small and focused on plugin lifecycle (loading, unloading, registering commands).
-   **Example file structure**:
    ```
    src/
      main.ts           # Plugin entry point, lifecycle management
      settings.ts       # Settings interface and defaults
      kanban-view.ts    # BasesView implementation for kanban
      commands/         # Command implementations
        command1.ts
        command2.ts
      ui/              # UI components, modals, views
        modal.ts
        view.ts
      utils/           # Utility functions, helpers
        helpers.ts
        constants.ts
      types.ts         # TypeScript interfaces and types
    ```
-   **Do not commit build artifacts**: Never commit `node_modules/`, `main.js`, or other generated files to version control.
-   Keep the plugin small. Avoid large dependencies. Prefer browser-compatible packages.
-   Generated output should be placed at the plugin root or `dist/` depending on your build setup. Release artifacts must end up at the top level of the plugin folder in the vault (`main.js`, `manifest.json`, `styles.css`).

## Manifest rules (`manifest.json`)

-   Must include (non-exhaustive):
    -   `id` (plugin ID; for local dev it should match the folder name)
    -   `name`
    -   `version` (Semantic Versioning `x.y.z`)
    -   `minAppVersion`
    -   `description`
    -   `isDesktopOnly` (boolean)
    -   Optional: `author`, `authorUrl`, `fundingUrl` (string or map)
-   Never change `id` after release. Treat it as stable API.
-   Keep `minAppVersion` accurate when using newer APIs.
-   Canonical requirements are coded here: https://github.com/obsidianmd/obsidian-releases/blob/master/.github/workflows/validate-plugin-entry.yml

## Testing

-   Manual install for testing: copy `main.js`, `manifest.json`, `styles.css` (if any) to:
    ```
    <Vault>/.obsidian/plugins/<plugin-id>/
    ```
-   Reload Obsidian and enable the plugin in **Settings → Community plugins**.

## Commands & settings

-   Any user-facing commands should be added via `this.addCommand(...)`.
-   If the plugin has configuration, provide a settings tab and sensible defaults.
-   Persist settings using `this.loadData()` / `this.saveData()`.
-   Use stable command IDs; avoid renaming once released.
-   **No default hotkeys**: Avoid setting default hotkeys for commands. Let users configure their own.
-   **Use appropriate callback type**:
    -   `callback`: Simple commands that always work
    -   `checkCallback`: Commands that may be disabled based on context
    -   `editorCallback`: Commands that operate on the editor

## Workspace, editor, and vault

-   **Avoid workspace.activeLeaf**: This is deprecated. Use `getActiveViewOfType()` instead:

```ts
const view = this.app.workspace.getActiveViewOfType(MarkdownView);
if (view) {
	// work with view
}
```

-   **Use Editor for active notes**: When editing the currently open note, use the `Editor` interface, not `Vault.modify()`.
-   **Use Vault.process() for closed notes**: For notes that aren't open, use `Vault.process()` instead of `Vault.modify()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ewerx/obsidian-bases-kanban](https://github.com/ewerx/obsidian-bases-kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
