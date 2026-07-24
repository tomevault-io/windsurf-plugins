---
trigger: always_on
description: Web Components library built with TypeScript. All components use Shadow DOM encapsulation, CSS variable theming, and SSR support.
---

# ranui — Component Library Reference

Web Components library built with TypeScript. All components use Shadow DOM encapsulation, CSS variable theming, and SSR support.

---

## Design Standards — read before building or changing any UI

**[docs/DESIGN.md](docs/DESIGN.md) is the authoritative, executable design standard.** Follow it whenever your work changes what a user sees. It is based on the Geist design system (light/dark only).

For each element's **attributes / properties / events / slots / `::part()`**, consult **[docs/COMPONENTS.md](docs/COMPONENTS.md)** (generated — run `npm run doc:api` after changing any component's API) and **[docs/style-tokens-public.md](docs/style-tokens-public.md)** for its CSS variables. The non-negotiables:

- **Color is a state ladder, not a palette.** Each scale step 100→1000 has one fixed job: 100 default bg · 200 hover bg · 300 active bg · 400 border · 500 hover border · 600 active border · 700 solid · 800 solid hover · 900 secondary text · 1000 primary text. Use the **semantic tokens** (`--ran-color-*`), never raw hex, in components.
- **Dark-safe fallbacks.** A component token's fallback must point at a token that _flips_ (`var(--ran-color-text, …)`, `var(--ran-gray-alpha-100, …)`, `var(--ran-blue-100, …)`) — never a light-only literal like `rgba(0,0,0,.06)` or `#e6f7ff`, which breaks in dark mode.
- **Spacing:** the `--ran-space-*` scale only (4px base, 9 values). 8 within a group, 16 between groups, 32–40 between sections.
- **Typography:** choose a role (heading / label / copy / button / mono), not a raw px size.
- **Elevation = role.** Pick the shadow by what the element _is_: in-flow surface (card/section) → `--ran-shadow-elevated`; floating overlay (dropdown, select, popover, toast/message) → `--ran-shadow-menu`; blocking dialog → `--ran-shadow-modal`. A floating overlay must never fall back to the card tier (`elevated`) — it looks flat.
- **Radius/motion:** use the tokens; prefer no motion (0ms) and keep what remains quick (150/200/300ms); respect `prefers-reduced-motion`.
- **Copy:** buttons = action + object ("Deploy project"); errors = what + how; toasts state the change ("Project deleted").
- **Accessibility:** WCAG AA contrast; never signal state by color alone (pair an icon/label); visible focus ring on every interactive element; icon-only controls need an `aria-label`; full keyboard nav.
- **Verify rendered output** in light _and_ dark, at narrow _and_ wide widths, across the materially changed states — code review alone is not enough.

---

## Project Layout

```
packages/ranui/
├── components/           # One component per directory
│   └── {name}/
│       ├── index.ts      # Component class + defineSSR()
│       └── index.less    # Shadow DOM styles (auto-imports base.less)
├── utils/
│   ├── component.ts      # ensureShadowRoot, ensureShadowElement, attribute helpers
│   ├── builder/          # ElementBuilder fluent DOM builder
│   ├── router/           # RouterCore, createRouter, useRouter, enableMpaViewTransitions
│   ├── i18n/             # I18nCore, createI18n, useI18n (framework-agnostic)
│   ├── ssr-registry.ts   # defineSSR, SSR support
│   ├── theme.ts          # setTheme, setThemeToken(s), initTheme (light/dark/system)
│   ├── style.ts          # adoptStyles, adoptSheetText
│   └── dom.ts            # falseList, isDisabled
├── theme/                # tokens.less (Geist base+semantic) + dark.less (dark mixin)
├── docs/DESIGN.md        # ⭐ AI-facing design standard — follow it for ANY UI work
├── docs/COMPONENTS.md    # ⭐ generated per-element API (attrs/props/events/slots/parts)
├── test/unit/            # *.contract.test.ts per component
├── demo/                 # Dev server entry (Vite); routed showcase (r-router)
├── index.ts              # Barrel exports + side-effect imports
├── theme.ts              # `ranui/theme` public entry (theming only, no components)
├── i18n.ts               # `ranui/i18n` public entry (i18n only, no components)
├── vite.config.ts        # Build + dev server config
├── vitest.config.ts      # Test config (jsdom, 80%+ coverage)
└── base.less             # Shared LESS variables/mixins (auto-imported)
```

---

## Component Architecture

### Canonical pattern

Every component follows this exact structure:

```typescript
import componentCss from './index.less?inline';
import { Div, EventManager, Slot } from '@/utils/builder';
import { RanElement } from '@/utils/index';
import {
  ensureShadowRoot,
  ensureShadowElement,
  getStringAttribute,
  setStringAttribute,
  syncSheetAttribute,
} from '@/utils/component';
import { defineSSR } from '@/utils/ssr-registry';

export class MyComponent extends RanElement {
  _events = new EventManager();
  _shadowDom!: ShadowRoot;
  _myEl!: HTMLElement; // store refs to queried elements

  static get observedAttributes(): string[] {
    return ['my-attr', 'sheet']; // always include 'sheet'
  }

  constructor() {
    super();
    this._shadowDom = ensureShadowRoot(this, componentCss);

    const root = ensureShadowElement(this._shadowDom, '.ran-mycomp', () =>
      Div().class('ran-mycomp').attr('part', 'mycomp').children(Slot()).build(),
    );

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaxus/ran](https://github.com/chaxus/ran) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
