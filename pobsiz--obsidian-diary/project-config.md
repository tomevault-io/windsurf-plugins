---
trigger: always_on
description: - Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
---

# Obsidian community plugin

## Project overview

- Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
- Source entry point: `src/main.ts`; esbuild bundles it to top-level `main.js`, which Obsidian loads.
- Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.
- Current plugin ID: `diary`.
- Current plugin version: `1.4.0`.
- Current minimum Obsidian version: `1.7.2`.

## Environment & tooling

- Node.js: use a supported LTS for local development. As of June 2026, Node.js 24 is the current LTS; this repo's CI validates Node.js `20.x` and `22.x`, and the release workflow currently builds with `18.x`.
- **Package manager: npm**. Use `npm ci` for clean CI-style installs and `npm install` when intentionally changing dependencies.
- **Bundler: esbuild** via `esbuild.config.mjs`; bundle all runtime dependencies into `main.js`.
- Types: `obsidian` type definitions.

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

- Use the repository script:

```bash
npm run lint
```

- The config uses `eslint-plugin-obsidianmd`, TypeScript ESLint, browser globals, and ignores generated `main.js`.
- Do not rely on a globally installed eslint for this repo.

## File & folder conventions

- **Organize code into multiple files**: Split functionality across separate modules rather than putting everything in `src/main.ts`.
- Source lives in `src/`. Keep `src/main.ts` focused on plugin lifecycle (loading, unloading, registering commands, registering views, wiring refresh events).
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

- Must include:
  - `id` (plugin ID; for local dev it should match the folder name)  
  - `name`  
  - `version` (Semantic Versioning `x.y.z`)  
  - `minAppVersion`  
  - `description`  
  - `author`
  - `isDesktopOnly` (boolean)  
- Optional: `authorUrl`, `fundingUrl` (string or map), `helpUrl`
- Never change `id` after release. Treat it as stable API.
- Keep `minAppVersion` accurate when using newer APIs.
- Keep the `manifest.json` version, `package.json` version, and `versions.json` mapping synchronized.
- Canonical requirements are coded here: https://github.com/obsidianmd/obsidian-releases/blob/master/.github/workflows/validate-plugin-entry.yml

## Testing

- Repository verification:

```bash
npm run build
npm run lint
```

- Manual install for testing: copy `main.js`, `manifest.json`, `styles.css` (if any) to:
  ```
  <Vault>/.obsidian/plugins/<plugin-id>/
  ```
- Reload Obsidian and enable the plugin in **Settings → Community plugins**.

## Commands & settings

- Any user-facing commands should be added via `this.addCommand(...)`.
- If the plugin has configuration, provide a settings tab and sensible defaults.
- Persist settings using `this.loadData()` / `this.saveData()`.
- Use stable command IDs; avoid renaming once released.

## Versioning & releases

- Bump `version` in `manifest.json` (SemVer) and update `versions.json` to map plugin version → minimum app version.
- This repo uses npm for version bumps. Prefer `npm version patch|minor|major --no-git-tag-version` so `package.json`, `package-lock.json`, `manifest.json`, and `versions.json` stay in sync through `version-bump.mjs`.
- Choose the smallest SemVer bump that accurately describes the release:
  - **Patch** (`x.y.z+1`): bug fixes, CSS/lint/build compatibility fixes, release metadata fixes, and other changes that should not alter user workflows.
  - **Minor** (`x.y+1.0`): user-visible features or improvements, new commands/settings/views, accessibility or mobile UX enhancements, and non-breaking behavior additions.
  - **Major** (`x+1.0.0`): breaking changes, removed or renamed commands/settings, incompatible data or file format changes, or a minimum Obsidian version increase that intentionally drops previously supported users.
- If a release contains mixed changes, use the highest applicable bump.
- When asked to push or release, inspect the diff first and state the selected version bump before committing.
- Create a GitHub release whose tag exactly matches `manifest.json`'s `version`. Do not use a leading `v`.
- Attach `manifest.json`, `main.js`, and `styles.css` (if present) to the release as individual assets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [POBSIZ/obsidian-diary](https://github.com/POBSIZ/obsidian-diary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
