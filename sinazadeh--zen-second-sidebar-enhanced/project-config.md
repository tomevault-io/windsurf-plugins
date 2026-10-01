---
trigger: always_on
description: Second Sidebar is a privileged Firefox and Zen Browser userChrome.js script loaded
---

# Repository guidance

## Project and runtime

Second Sidebar is a privileged Firefox and Zen Browser userChrome.js script loaded
through fx-autoconfig, [Sine](https://github.com/CosmoCreeper/Sine) (via the
`theme.json` manifest at the repo root), or a compatible script loader. It adds
a second sidebar and web panels to the browser UI. This repository is adapted
for **Zen Browser** while maintaining compatibility with standard Firefox. It
is not a WebExtension or a Node.js/web application: there is no bundler,
development server, or build step. Deploy the contents of `src/` as-is (for
Sine, `theme.json` does this automatically - see "Loader portability" below).

Read `README.md` for features and installation, and the relevant implementation
before changing behavior. Follow applicable user-level agent instructions;
keep machine-specific subagent configuration outside this repository.

## Code map

All paths below are relative to `src/second_sidebar/`, except the entry point.

| Location                                       | Responsibility                                                                            |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `src/second_sidebar.uc.mjs`                    | Waits for Firefox/Zen startup, skips nested panel windows, injects and decorates sidebar. |
| `sidebar_injector.mjs`                         | Loads settings/state, creates elements and controllers, then applies settings/state.      |
| `sidebar_elements.mjs`, `browser_elements.mjs` | Sidebar element registry and access to existing browser chrome elements.                  |
| `sidebar_controllers.mjs`                      | Creates and connects controllers in dependency order.                                     |
| `controllers/`                                 | Sidebar/panel behavior, geometry, shortcuts, popup actions, and cross-window events.      |
| `xul/`, `xul/base/`                            | UI components and shared fluent wrappers around XUL/HTML elements.                        |
| `css/`, `sidebar_decorator.mjs`                | CSS template-string exports, combined and injected into the chrome document.              |
| `settings/`                                    | Defaults, serialization, persisted settings, and panel state.                             |
| `wrappers/`                                    | Adapters for privileged Firefox/Gecko globals and services.                               |
| `patchers/`                                    | Compatibility patches for Firefox/Zen UI implementation.                                  |
| `patchers/source_patches.mjs`                  | Text patches for Firefox sources; browser-global-free so Node tests can run them.         |
| `utils/browser_layout.mjs`                     | Browser container resolution (`#zen-tabbox-wrapper` for Zen, `#browser` for Firefox).     |
| `utils/`, `icons/`                             | Shared helpers and SVG assets.                                                            |
| `tests/` (repo root)                           | Node unit tests for pure logic (settings, import/export, source patches).                 |
| `scripts/` (repo root)                         | Development scripts, e.g. `check_patch_targets.mjs` (see "Static checks").                |

## Implementation conventions

- Use ES modules with explicit relative `.mjs` imports. Follow the existing
  two-space indentation, double quotes, semicolons, and Prettier formatting.
  Files use `snake_case`; classes use `PascalCase`; methods use `camelCase`.
- Keep JSDoc consistent with nearby code. Some imports exist only for JSDoc and
  use a targeted `no-unused-vars` suppression; do not remove their type context
  just to silence lint.
- Put behavior in controllers, UI construction in `xul/`, and Firefox API access
  in the corresponding wrapper. Reuse `XULElement` and `utils/xul.mjs` helpers.
  Preserve the XUL/HTML element distinction.
- Support both Zen Browser and standard Firefox layout hierarchies. Avoid hardcoding
  `#browser` when attaching or sizing wrappers; use `requireBrowserContainerElement()`
  or selectors targeting `#zen-tabbox-wrapper, #browser`.
- Use existing `sb2-` IDs/classes and `--sb2-` CSS variables for new sidebar
  styles. In Zen Browser, adhere to `--sb2-zen-*` theme variables, `--zen-border-radius`,
  `--zen-element-separation`, and `--zen-colors-*`. Preserve `:root:has(#zen-tabbox-wrapper)`
  and `[zen-right-side="true"]` rules. Add new CSS exports to `sidebar_decorator.mjs`
  when they need to be injected.
- Match native theme tokens and controls. Keep keyboard focus, shortcuts, tooltips,
  and both sidebar positions working across both browsers.
- When uncollapsing the sidebar in controllers, remove inline margin properties
  (`removeProperty("margin-right")` / `removeProperty("margin-left")`) rather than
  forcing `0px`, allowing Zen's flex/grid layout engine to position adjacent content correctly.
- `MozButton` and `Toggle` wrap HTML custom elements with `isXUL: false`;
  popup/menu wrappers use XUL. Reuse their factories when adding controls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinazadeh/zen-second-sidebar-enhanced](https://github.com/sinazadeh/zen-second-sidebar-enhanced) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
