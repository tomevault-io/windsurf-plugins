---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Repository Overview

This is **zorilla**, a maintained fork of puppeteer-extra - a modular plugin framework that extends Puppeteer and Playwright with additional functionality through a clean plugin system. The project consists of two main frameworks (`@zorilla/puppeteer-extra` and `@zorilla/playwright-extra`) and a collection of plugins that work with both.

### Key Architecture Concepts

**Plugin System Architecture:**
- Plugins extend `PuppeteerExtraPlugin` base class from `packages/puppeteer-extra-plugin`
- Plugins are framework-agnostic and work with both Puppeteer and Playwright through a compatibility shim
- Plugins hook into lifecycle events: `beforeLaunch`, `beforeConnect`, `onBrowser`, `onPageCreated`, `onContextCreated`, etc.
- Plugins can declare requirements (`headful`, `runLast`, `dataFromPlugins`) and dependencies on other plugins
- Plugin options are deep-merged with defaults and accessible via `this.opts`

**Puppeteer vs Playwright:**
- `packages/puppeteer-extra`: Wraps Puppeteer API, supports `puppeteer` and `puppeteer-core`
- `packages/playwright-extra`: Wraps Playwright API, supports all browsers (chromium, firefox, webkit)
- Both use the same plugin base class with a compatibility shim in `packages/playwright-extra/src/puppeteer-compatiblity-shim`
- Playwright-extra uses `addExtra()` internally to create separate instances per browser type

**Dual Export Pattern:**
- Default export: Drop-in replacement that auto-requires installed puppeteer/playwright
- `addExtra()` export: Allows creating custom instances with different plugins or wrapping alternative implementations

## Development Commands

### Building
```bash
pnpm install                           # Install all dependencies
pnpm build                             # Build all packages in sequence
cd packages/<package-name> && pnpm build  # Build single package
```

### Testing
```bash
pnpm test                              # Run all tests across all packages
pnpm -r run test:coverage             # Run tests with coverage for all packages
cd packages/<package-name> && pnpm test           # Test single package
cd packages/<package-name> && pnpm test:coverage  # Test with coverage
```

**Testing Notes:**
- Most packages use Vitest with v8 coverage provider
- `playwright-extra` uses `@playwright/test` with c8 for coverage
- Stealth plugin tests run with `--pool.threads.maxThreads=2` to avoid race conditions
- Coverage reports are generated in `coverage/` directories (gitignored)

### Linting & Formatting
```bash
pnpm check                             # Check code with Biome (no changes)
pnpm fix                               # Auto-fix issues with Biome
```

Uses Biome (not ESLint/Prettier) for linting and formatting.

### Publishing
```bash
pnpm changeset                         # Create a changeset for your changes
pnpm version                           # Version packages and update changelogs
pnpm release                           # Build and publish to npm
```

Uses Changesets for version management. Lerna is no longer used.

## Monorepo Structure

**Package Categories:**
1. **Core Frameworks:** `puppeteer-extra`, `playwright-extra`
2. **Plugin Base:** `puppeteer-extra-plugin` (used by all plugins)
3. **Plugins:** All packages prefixed with `puppeteer-extra-plugin-*`
4. **Utilities:** `plugin-proxy-router`, `extract-stealth-evasions`

**Important Plugin Packages:**
- `puppeteer-extra-plugin-stealth`: Complex evasion system with sub-evasions in `src/evasions/`
- `puppeteer-extra-plugin-recaptcha`: Integrates with captcha solving services
- `puppeteer-extra-plugin-adblocker`: Uses `@cliqz/adblocker-puppeteer`

## Plugin Development

### Creating a New Plugin

1. Extend `PuppeteerExtraPlugin` from `@zorilla/puppeteer-extra-plugin`
2. Implement required getter `name` (convention: matches package name without prefix)
3. Optional: Implement `defaults` getter for default options
4. Optional: Implement lifecycle hooks (`onPageCreated`, `beforeLaunch`, etc.)
5. Export a factory function that returns a new plugin instance

```typescript
import { PuppeteerExtraPlugin } from '@zorilla/puppeteer-extra-plugin'

class MyPlugin extends PuppeteerExtraPlugin {
  get name() { return 'my-plugin' }

  get defaults() { return { option: 'default' } }

  async onPageCreated(page) {
    // Plugin logic using this.opts
  }
}

export default (opts) => new MyPlugin(opts)
```

### Plugin Lifecycle Events

**Browser Lifecycle:**
- `beforeLaunch(options)` / `beforeConnect(options)` - Modify launch/connect options
- `afterLaunch(browser, options)` / `afterConnect(browser, options)` - Post-launch actions
- `onBrowser(browser, options)` - Called for any browser acquisition
- `onDisconnected()` - Browser disconnected

**Page Lifecycle:**
- `beforeContext(options)` - Before context creation (Playwright)
- `onContextCreated(context)` - Context created (Playwright)
- `onPageCreated(page)` - New page created
- `onTargetCreated(target)` - New target created

### Debugging Plugins

```bash
DEBUG=puppeteer-extra*,puppeteer-extra-plugin:* node script.js
DEBUG=playwright-extra*,puppeteer-extra-plugin:* node script.js
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zorillajs/zorilla](https://github.com/zorillajs/zorilla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
