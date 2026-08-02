---
trigger: always_on
description: - Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
---

# Obsidian community plugin

## Project overview

- Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
- Entry point: `src/main.ts` compiled to `main.js` and loaded by Obsidian.
- Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.

## Environment & tooling

- Node.js: use current LTS (Node 18+ recommended).
- **Package manager: npm** (required - `package.json` defines npm scripts and dependencies).
- **Bundler: esbuild** (required - `esbuild.config.mjs` and build scripts depend on it).
- Types: `obsidian` type definitions.

### Install

```bash
npm install
```

### Dev (watch)

```bash
npm run dev:esbuild
```

### Production build

```bash
npm run build:esbuild
```

## Linting

- ESLint is preconfigured.
- Run `npm run lint` (if available) to lint the project.

## File & folder conventions

- Source lives in `src/`. Keep `main.ts` small and focused on plugin lifecycle (loading, unloading, registering commands).
- **Current structure**:
    ```
    src/
      main.ts                        # Plugin entry point, lifecycle management
      types.ts                       # TypeScript interfaces and types
      constants.ts                   # Default settings, static data
      l10n.ts                        # Localization (t() function)
      util.tsx                       # Utility functions and Preact helpers
      manager/commands/              # Feature managers (one per UI location)
        commandManager.ts            # Abstract base class
        leftRibbonManager.ts
        statusBarManager.ts
        pageHeaderManager.ts
        menuManager.ts
        explorerManager.ts
        textToolbarManager.ts        # Text selection toolbar (new)
        index.ts
      ui/
        settingTab.ts                # Registers settings tab with Obsidian
        settingTabModal.ts
        icons.ts
        addCommandModal.ts
        chooseIconModal.ts
        chooseCustomNameModal.ts
        confirmDeleteModal.ts
        components/                  # Preact components
          settingTabComponent.tsx    # Tab container, tab list
          commandViewerComponent.tsx # Reusable command list with add/remove/reorder
          commandComponent.tsx       # Single command row
          hidingViewer.tsx           # Eye-toggle accordions for hiding native items
          settingComponent.tsx       # ToggleComponent, SliderComponent, EyeToggleComponent
          TextToolbarSettings.tsx    # Text toolbar tab content (new)
          AdvancedToolbarSettings.tsx
          MacroViewer.tsx
          MacroBuilder.tsx
          Accordion.tsx
          About.tsx
      styles/
        styles.scss
        advanced-toolbar.scss
    locale/                          # Translation JSON files (en.json is canonical)
    ```
- **Do not commit build artifacts**: Never commit `node_modules/`, `main.js`, or other generated files.

## Key architectural patterns

### Manager pattern
Each UI location (ribbon, status bar, page header, editor menu, etc.) has a manager class that extends `CommandManagerBase`. Managers:
- Own the `CommandIconPair[]` array for their location
- Implement `addCommand`, `removeCommand`, `reorder`
- Handle DOM creation and updates for their location
- Are initialized in `main.ts` `onload()` and stored on `plugin.manager`

### Settings storage
Settings are persisted via `plugin.loadData()` / `plugin.saveData()`. The `CommanderSettings` interface in `types.ts` is the source of truth. Always add a nullish guard in `onload()` for any new optional array field so that users upgrading from older versions don't get errors:
```typescript
this.settings.hide.textToolbar ??= [];
```

### Localization
All user-visible strings should go through `t()` from `src/l10n.ts`. The canonical locale is `locale/en.json`. If you add a new UI string, add it to `en.json` (and ideally other locale files). Strings not in the locale fall back to `en.json`; if not found there either, `t()` returns `undefined`, which renders as blank text.

### Preact UI
The settings UI uses Preact (not React). Components live in `src/ui/components/`. Reuse existing components:
- `ToggleComponent` / `SliderComponent` / `EyeToggleComponent` from `settingComponent.tsx`
- `CommandViewer` from `commandViewerComponent.tsx` — takes a `CommandManagerBase`, renders the command list with add/remove/reorder
- `Accordion` from `Accordion.tsx`

To add a new settings tab:
1. Create a Preact component in `src/ui/components/`
2. Add an entry to the `tabs` array in `settingTabComponent.tsx`
3. The tab name must be a plain string (not `undefined`) — if using `t()`, ensure the key exists in `en.json`

## Adding a new feature (checklist)

1. **Types**: Add fields to `CommanderSettings` in `src/types.ts`
2. **Defaults**: Add corresponding defaults in `DEFAULT_SETTINGS` in `src/constants.ts`
3. **Manager**: Create `src/manager/commands/<feature>Manager.ts` extending `CommandManagerBase`; export from `index.ts`
4. **main.ts**: Add nullish guard for any new array fields; instantiate manager; add to `plugin.manager`
5. **Settings UI**: Create component in `src/ui/components/`; add tab to `settingTabComponent.tsx`
6. **Locale**: Add any new UI strings to `locale/en.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsmorabito/obsidian-commander](https://github.com/jsmorabito/obsidian-commander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
