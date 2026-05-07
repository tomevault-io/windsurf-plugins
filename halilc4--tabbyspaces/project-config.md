---
trigger: always_on
description: Visual split-layout workspace editor for Tabby.
---

# TabbySpaces

Visual split-layout workspace editor for Tabby.

## Git Workflow

- **main** - Stable releases only. Do not commit directly.
- **dev** - Active development. All work happens here.

```bash
# Normal workflow
git checkout dev        # Work on dev
# ... make changes ...
git commit

# Release workflow
git checkout main
git merge dev
git tag v0.x.0
git push --tags
git checkout dev        # Back to work
```

## Tech Stack

- **Framework**: Angular 15 (Tabby uses Angular 15)
- **Language**: TypeScript 4.9
- **Templates**: Pug (.pug)
- **Styles**: SCSS
- **Build**: Webpack 5

## Structure

```
src/
├── index.ts                 # NgModule entry point
├── build-config.ts          # Build-time constants (CONFIG_KEY, DISPLAY_NAME)
├── models/                  # TypeScript interfaces
│   └── workspace.model.ts
├── services/                # Business logic
│   ├── workspaceEditor.service.ts
│   └── startupCommand.service.ts
├── providers/               # Tabby config/settings/toolbar providers
│   ├── config.provider.ts
│   ├── settings.provider.ts
│   └── toolbar.provider.ts
├── styles/                  # Shared SCSS (modular DRY)
│   ├── _variables.scss      # Spacing, radius, colors, z-index
│   └── _mixins.scss         # Reusable patterns
└── components/              # Angular components (.ts, .pug, .scss)
    ├── workspaceList        # Main settings UI
    ├── workspaceEditor      # Single workspace editor
    ├── paneEditor           # Pane configuration
    └── splitPreview         # Visual split preview
```

## Styles

Modular DRY SCSS architecture. All components load shared styles via `@use '../styles/index' as *;`.

- **Variables**: `$spacing-*`, `$radius-*`, `$color-*`, `$z-*`, `$transition-*`
- **Mixins**: Layout, form, card, and button patterns. See `src/styles/_mixins.scss` for the available mixins.
- **Theming**: Uses Tabby's `--theme-*` CSS variables

See `docs/DESIGN.md` for details.

## Build

```bash
npm install            # .npmrc has legacy-peer-deps=true
npm run build          # Production build → dist/
npm run build:dev      # Dev build → dist-dev/ (isolated package)
npm run watch:dev      # Watch mode for dev build
```

Debug: `Ctrl+Shift+I` in Tabby opens DevTools.

## Tabby Plugin Patterns

### package.json (required)
```json
{
  "keywords": ["tabby", "tabby-plugin"],
  "main": "dist/index.js",
  "tabbyPlugin": {
    "name": "tabbyspaces",
    "displayName": "TabbySpaces",
    "description": "Workspaces for Tabby - Visual split-layout workspace editor"
  }
}
```

### Config Provider
```typescript
@Injectable()
export class MyConfigProvider extends ConfigProvider {
  defaults = { myPlugin: { setting: 'value' } }
}
```

### Settings Tab Provider
```typescript
@Injectable()
export class MySettingsProvider extends SettingsTabProvider {
  id = 'my-plugin'
  icon = 'cog'
  title = 'My Plugin'
  getComponentType() { return MySettingsComponent }
}
```

### Module Registration
```typescript
@NgModule({
  providers: [
    { provide: ConfigProvider, useClass: MyConfigProvider, multi: true },
    { provide: SettingsTabProvider, useClass: MySettingsProvider, multi: true },
  ],
})
export default class MyModule {}
```

## Data Model

- `Workspace` - Main object with name, icon, color, root split, launchOnStartup
- `WorkspaceSplit` - Recursive structure with orientation, ratios, children
- `WorkspacePane` - Leaf node with profileId (reference to existing Tabby profile), cwd, startupCommand, title

### Workspace Fields
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Display name |
| `icon` | string | FontAwesome icon name (without fa- prefix) |
| `color` | string | Hex color |
| `root` | WorkspaceSplit | Root split node |
| `launchOnStartup` | boolean | Auto-open when Tabby starts (multiple allowed) |

## Architecture

### Storage
Plugin stores workspaces in `config.store.tabbyspaces.workspaces`. No profiles are generated in `config.store.profiles`.

### Opening Workspaces
1. Generate temporary `split-layout` recovery token from workspace model (includes `options.cwd`)
2. Open via `ProfilesService.openNewTabForProfile()`
3. `StartupCommandService` listens for `tabOpened$` events
4. Match terminal tabs by pane ID (passed via `tabCustomTitle`)
5. Send startup command via `sendInput()` (if defined)

### CWD Handling
CWD is set via native `options.cwd` in the recovery token. The shell spawns directly in the target directory - no visible `cd` commands.

### Profile Support
Plugin supports both user-defined profiles (`type: 'local'`) and built-in shells (`type: 'local:cmd'`, `'local:powershell'`, `'local:wsl'`, etc.). Profile lookup uses a two-stage approach:
1. First checks user profiles in `config.store.profiles`
2. Falls back to cached profiles from `profilesService.getProfiles()` (includes built-ins)

### Launch on Startup
Workspaces with `launchOnStartup: true` are automatically opened when Tabby starts. Multiple workspaces can be marked. Logic is in `toolbar.provider.ts` constructor with 500ms delay to ensure Tabby is ready.

### Migration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [halilc4/tabbyspaces](https://github.com/halilc4/tabbyspaces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
