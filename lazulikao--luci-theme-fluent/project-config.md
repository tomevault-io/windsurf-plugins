---
trigger: always_on
description: **FluentUI 2 theme for OpenWrt LuCI** — standalone repo (not monorepo). Built with SCSS, ucode templates, CSS custom properties for light/dark/auto theming.
---

# AGENTS.md - luci-theme-fluent Developer Guide

**FluentUI 2 theme for OpenWrt LuCI** — standalone repo (not monorepo). Built with SCSS, ucode templates, CSS custom properties for light/dark/auto theming.

- **Repo**: `LazuliKao/luci-theme-fluent` (separate, not `luci-theme-argon`)
- **Branch**: `openwrt-24.10`, `main`
- **Targets**: OpenWrt 24.10.7 (opkg/ipk), OpenWrt 25.12.4 (apk)

## Quick Start

```bash
pnpm install          # install deps from root + src/ (pnpm workspace)
pnpm run build        # compile SCSS + LuCI JS/TSX (runs "cd src && pnpm run build")
pnpm run watch        # auto-rebuild both
pnpm run lint         # Biome lint on package/luci-theme-fluent/htdocs/ + src/web/resources/
pnpm run typecheck    # "cd src && tsc -p tsconfig.json --noEmit"
```

## Build System (Rsbuild)

The project uses **Rsbuild** (not raw sass CLI) configured in `src/rsbuild.config.ts` with two environments:

| Environment | Entry | Output | Notes |
|---|---|---|---|
| `css` | `src/scss/fluent.scss` | `package/luci-theme-fluent/htdocs/luci-static/fluent/css/fluent.css` | Sass via `@rsbuild/plugin-sass`, SVG inlining via `dataUriLimit: MAX_SAFE_INTEGER`. Custom plugin removes generated `fluent.js`. Minify: off. |
| `js` | `src/web/resources/{menu-fluent.tsx, view/fluent-config.tsx}` | `package/luci-theme-fluent/htdocs/luci-static/resources/{menu-fluent.js, view/fluent-config.js}` | React JSX via `@lazulikao/luci-types`, LuCI `require` preamble via `BannerPlugin`, `return main;` footer. Minify: on (but splitChunks/runtimeChunk off). |

**Key rsbuild quirks**:

- JS env has `rspack.BannerPlugin` prepending `"require baseclass"` / `"require ui"` and appending `return main;`.
- CSS env has `RemoveEntryJsPlugin` that deletes `fluent.js` from output.
- `splitChunks: false`, `runtimeChunk: false`, `minimize: false` in JS env.

## Project Structure

```
luci-theme-fluent/
├── package/
│   └── luci-theme-fluent/
│       ├── htdocs/luci-static/
│       │   ├── fluent/css/fluent.css       # Compiled CSS (NOT committed? check .gitignore)
│       │   ├── fluent/background/          # User-uploaded backgrounds
│       │   ├── fluent/fonts/               # Empty directory
│       │   ├── fluent/icon/                # favicon.ico, icon-192.png, favicon-32.png, manifest.json
│       │   ├── fluent/img/fluent.svg       # Theme logo
│       │   └── resources/                  # Compiled JS: menu-fluent.js, view/fluent-config.js
│       ├── ucode/template/themes/fluent/   # 6 ucode templates (header.ut, footer.ut, header_login.ut, footer_login.ut, out_header_login.ut, sysauth.ut)
│       ├── root/
│       │   ├── etc/config/fluent           # Default UCI config (40+ options)
│       │   ├── etc/uci-defaults/luci-fluent # Theme registration + default config initialization
│       │   ├── usr/libexec/fluent/online_wallpaper  # Shell script: fetches Bing/Unsplash login backgrounds
│       │   ├── usr/libexec/rpcd/luci.fluent        # RPC daemon: list/remove/rename background files
│       │   ├── usr/share/luci/menu.d/luci-theme-fluent.json  # Menu registration for config view
│       │   └── usr/share/rpcd/acl.d/luci-theme-fluent.json   # ACL permissions: fluent UCI + background file access
│       ├── po/
│       │   ├── templates/fluent.pot        # POT template (66 strings)
│       │   └── zh_Hans/fluent.po           # Simplified Chinese translations
│       └── Makefile                        # OpenWrt package definition
├── src/
│   ├── scss/
│   │   ├── fluent.scss            # Entry point (47 @use imports)
│   │   ├── _variables.scss        # Design tokens (157 lines: typography, spacing, radius, z-index, brand ramps)
│   │   ├── _mixins.scss           # Responsive breakpoints, button/input/card/table/scrollbar mixins (394 lines)
│   │   ├── _icons.scss            # 15 FluentUI SVG icons as SCSS variables, fluent-icon() + fluent-icon-content() mixins
│   │   ├── _base.scss             # CSS reset, typography, animations (437 lines)
│   │   ├── components/            # 24 component partials
│   │   │   ├── _buttons, _inputs, _textarea, _select, _checkboxes
│   │   │   ├── _tables, _cards, _tabs, _navigation, _dropdown
│   │   │   ├── _dynlist, _password, _modals, _progress, _scrollbars
│   │   │   ├── _errors, _alert-message, _cbi-forms, _cbi-dialogs
│   │   │   ├── _cbi-network, _cbi-widgets, _dashboard, _menu-button, _tooltips
│   │   ├── layouts/               # _login, _sidebar, _header, _main (4 files)
│   │   ├── themes/                # _light.scss, _dark.scss
│   │   └── overrides/             # Plugin-specific SCSS overrides
│   │       ├── index.scss         # @forward dispatcher (manual maintenance)
│   │       ├── overrides-utils    # Shared override utilities
│   │       ├── luci-mod-dashboard, luci-app-firewall
│   │       ├── system-channel_analysis, admin-status-realtime
│   │       └── README.md          # How to add new overrides (page-scoped, body.page-* selector)
│   ├── web/
│   │   ├── index.ts               # Just declares baseclass + ui types
│   │   └── resources/
│   │       ├── menu-fluent.tsx     # Entry: renders sidebar nav + tab menus via LuCI menu API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LazuliKao/luci-theme-fluent](https://github.com/LazuliKao/luci-theme-fluent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
