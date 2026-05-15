---
trigger: always_on
description: Per official Obsidian plugin guidelines:
---

# adapt-to-current-view

## Guidelines

Per official Obsidian plugin guidelines:

- **No `FileSystemAdapter` cast**: Gate ALL `FileSystemAdapter` usage behind `instanceof` check. Mobile uses `CapacitorAdapter`.
- **No `process.platform`**: Use Obsidian's `Platform` API instead.
- **Settings headings**: ONLY use section headings if there are multiple sections. General settings go at top without heading.
- **No `insertAdjacentHTML`**: Use DOM API or Obsidian helpers (`createEl()`, `createDiv()`, `createSpan()`) instead.
- **Use `el.empty()`**: To clean up HTML element contents.
- **Correct callback type**: `callback` for unconditional, `checkCallback` for conditional, `editorCallback`/`editorCheckCallback` when an active editor is required.
- **No `workspace.activeLeaf`**: Use `getActiveViewOfType()` instead.
- **Null-check `activeEditor`**: Use optional chaining (`activeEditor?.editor`).
- **Editor API for active file**: Prefer over `Vault.modify()` (preserves cursor, selection, folded state).
- **`Vault.process()` for background edits**: Atomic operation, avoids conflicts vs. `Vault.modify()`.
- **`FileManager.processFrontMatter()`**: NEVER parse/modify YAML manually.
- **Vault API over Adapter API**: Better performance (caching) and safety (serial operations).
- **`normalizePath()`**: ALWAYS use for human-defined paths (handles slashes, spaces, Unicode, cross-platform).
- **`updateOptions()`**: To change or reconfigure editor extensions after registration (updates ALL editors).
- **No overriding core styling**: Add own classes and scope styling to them.
- **Obsidian CSS variables**: Use for consistent styling. Create custom variables ONLY if no matching variable exists.
- **CSS variable fallbacks**: ALWAYS include a fallback value for external (Obsidian-provided) CSS variables: `var(--icon-s, 18px)` NOT `var(--icon-s)`. Use default theme values. Plugin-owned and `--size-*` variables are exempt.
- **`instanceof` before casting**: Test before casting to `TFile`, `TFolder`, `FileSystemAdapter`, etc.
- **Optimize load time**: Initial UI setup on `workspace.onLayoutReady()`, NOT in constructor or `onload()`.
- **Deferred views**: Tabs load as `DeferredView` until visible. NEVER assume `leaf.view` is the real view — use `instanceof`. `await revealLeaf(leaf)` or `await leaf.loadIfDeferred()` before access.
- **License**: Include LICENSE file. Comply with original licenses of used code. Attribute in README if required.
- **Trademark**: NEVER use "Obsidian" in a way that suggests the plugin is first-party.

***

## Project overview

- Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
- Entry point: `main.ts` compiled to `main.js` and loaded by Obsidian.
- Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.

## Environment & tooling

- Node.js: use current LTS (Node 18+ recommended).
- **Package manager: npm** (required for this sample - `package.json` defines npm scripts and dependencies).
- **Bundler: esbuild** (required for this sample - `esbuild.config.mjs` and build scripts depend on it). Alternative bundlers like Rollup or webpack are acceptable for other projects if they bundle all external dependencies into `main.js`.
- Types: `obsidian` type definitions.

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

- To use eslint install eslint from terminal: `npm install -g eslint`
- To use eslint to analyze this project use this command: `eslint main.ts`
- eslint will then create a report with suggestions for code improvement by file and line number.
- If your source code is in a folder, such as `src`, you can use eslint with this command to analyze all files in that folder: `eslint ./src/`

## File & folder conventions

- **Organize code into multiple files**: Split functionality across separate modules rather than putting everything in `main.ts`.
- Source lives in `src/`. Keep `main.ts` small and focused on plugin lifecycle (loading, unloading, registering commands).
- **Example file structure**:
  ```
  src/
    main.ts           # Plugin entry point, lifecycle management
    settings.ts       # Settings interface and defaults
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
- **Do not commit build artifacts**: Never commit `node_modules/`, `main.js`, or other generated files to version control.
- Keep the plugin small. Avoid large dependencies. Prefer browser-compatible packages.
- Generated output should be placed at the plugin root or `dist/` depending on your build setup. Release artifacts must end up at the top level of the plugin folder in the vault (`main.js`, `manifest.json`, `styles.css`).

## Manifest rules (`manifest.json`)

- Must include (non-exhaustive):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [churnish/adapt-to-current-view](https://github.com/churnish/adapt-to-current-view) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
