---
trigger: always_on
description: This document provides context and guidelines for AI coding assistants working on this codebase.
---

# AI Agent Instructions for OpenShift Console Plugin Template

This document provides context and guidelines for AI coding assistants working on this codebase.

## Project Overview

This is a **template repository** for creating OpenShift Console dynamic plugins. It's meant to be used via GitHub's "Use this template" feature, NOT forked. The template provides a minimal starting point for extending the OpenShift Console UI with custom pages and functionality.

> **⚠️ WARNING:**
> This repository is used by multiple large-scale enterprise web applications. Please proceed with caution when making any changes to this codebase. Changes here can affect downstream projects that depend on this template.
>
> **Only make changes that should be standard practice for ALL plugins created from this template.** If a change is specific to one plugin use case, it belongs in the instantiated plugin repository, not in this template.

**Key Technologies:**
- TypeScript + React 18
- PatternFly 6 (UI component library)
- Rspack with Module Federation
- react-i18next for internationalization
- Playwright for e2e testing
- Helm for deployment

**Compatibility:** Requires OpenShift 4.12+ (uses ConsolePlugin CRD v1 API)

## Architecture & Patterns

### Dynamic Plugin System

This plugin uses module federation to load at runtime into the OpenShift Console. Key files:

- `console-extensions.json`: Declares what the plugin adds to console (routes, nav items, etc.)
- `package.json` `consolePlugin` section: Plugin metadata and exposed modules mapping
- `rspack.config.ts`: Configures module federation and build

**Critical:** Any component referenced in `console-extensions.json` must have a corresponding entry in `package.json` under `consolePlugin.exposedModules`.

### Component Structure

- Use functional components with hooks (NO class components)
- All components should be TypeScript (`.tsx`)
- Follow PatternFly component patterns
- Use PatternFly CSS variables instead of hex colors (dark mode compatibility)

### Styling Constraints

**IMPORTANT:** The `.stylelintrc.yaml` enforces strict rules to prevent breaking console:

- **NO hex colors** - use PatternFly CSS variables (e.g., `var(--pf-v6-global-palette--blue-500)`)
- **NO naked element selectors** (like `table`, `div`) - prevents overwriting console styles
- **NO `.pf-` or `.co-` prefixed classes** - these are reserved for PatternFly and console
- **Prefix all custom classes** with plugin name (e.g., `console-plugin-template__nice`)

Don't disable these rules without understanding they protect against layout breakage!

## Internationalization (i18n)

**Namespace Convention:** `plugin__<plugin-name>` (e.g., `plugin__console-plugin-template`)

### In React Components:
```tsx
const { t } = useTranslation('plugin__console-plugin-template');
return <h1>{t('Hello, World!')}</h1>;
```

### In console-extensions.json:
```json
"name": "%plugin__console-plugin-template~My Label%"
```

**After adding/changing messages:** Run `yarn i18n` to update locale files in `/locales`

## File Organization

```
src/
  components/          # React components
    ExamplePage.tsx   # Example page component
    *.css            # Component styles (scoped with plugin prefix)
console-extensions.json # Plugin extension declarations
package.json           # Plugin metadata in consolePlugin section
tsconfig.json          # TypeScript config (strict: false currently)
rspack.config.ts      # Module federation + build config
locales/               # i18n translation files
charts/                # Helm chart for deployment
integration-tests/     # Playwright e2e tests
```

## Development Workflow

### Local Development
1. `yarn install` - install dependencies
2. `yarn start` - starts dev server on port 9001 with CORS
3. `yarn start-console` - runs OpenShift console in container (requires cluster login)
4. Navigate to http://localhost:9000/example

### Code Quality
- `yarn lint` - runs eslint, prettier, and stylelint (with --fix)
- Linting is mandatory before commits
- Follow existing code patterns in the repo

### Testing
- `yarn test` - runs Jest unit tests
- `yarn test-e2e` - opens Playwright in headed mode
- `yarn test-e2e-headless` - runs Playwright in headless mode
- Add e2e tests for new pages/features

## TypeScript Configuration

Current config has `strict: true` and enforces:
- `noUnusedLocals: true`
- All files should use `.tsx` extension

## Common Development Tasks

### Adding a New Page
1. Create component in `src/components/MyPage.tsx`
2. Add to `package.json` `exposedModules`: `"MyPage": "./components/MyPage"`
3. Add route in `console-extensions.json`:
   ```json
   {
     "type": "console.page/route",
     "properties": {
       "path": "/my-page",
       "component": { "$codeRef": "MyPage" }
     }
   }
   ```
4. Optional: Add nav item in `console-extensions.json`
5. Run `yarn i18n` if you added translatable strings

### Adding a Navigation Item
```json
{
  "type": "console.navigation/href",
  "properties": {
    "id": "my-nav-item",
    "name": "%plugin__console-plugin-template~My Page%",
    "href": "/my-page",
    "perspective": "admin",
    "section": "home"
  }
}
```

### Updating Plugin Name
When instantiating from template, update:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/console-plugin-template](https://github.com/openshift/console-plugin-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
