---
trigger: always_on
description: > **Stack**: Laravel Blade + Tailwind v4 + daisyUI v5 + Alpine.js. This is the Blade stack adapter for Pinion UI. Vanilla / React / Vue / Web Components adapters will each ship as their own NPM package with their own `AGENTS.md` (planned v0.5+). One package = one stack adapter, self-contained.
---

# AGENTS.md — pinion-ui (Laravel Blade adapter)

> **Stack**: Laravel Blade + Tailwind v4 + daisyUI v5 + Alpine.js. This is the Blade stack adapter for Pinion UI. Vanilla / React / Vue / Web Components adapters will each ship as their own NPM package with their own `AGENTS.md` (planned v0.5+). One package = one stack adapter, self-contained.

**Read this file before writing code that uses `sparrowhawk-labs/pinion-ui`.** It captures the rules, gotchas, and lookup paths an AI agent needs to use Pinion UI correctly.

## What this package is

A Laravel Blade component library (Tailwind v4 + daisyUI v5 + Alpine.js). 46 components — see [`reference/components/index.md`](./reference/components/index.md).

The package also ships the **mermaid/SVG color glue** as build artifacts in [`dist/`](./dist/): `dist/mermaid-theme.js` (runtime JS — inline it inside a `type="module"` mermaid block so figures follow `data-theme`/`data-tune`) and `dist/mermaid-theme.json` (expanded color/theme data for server-side SVG rendering). Canon + regen live in [`scripts/mermaid-theme/`](./scripts/mermaid-theme/) (`node scripts/mermaid-theme/build.mjs`).

## Calling convention

- **Anonymous (default)**: `<x-button>`, `<x-modal>`, `<x-tabs>`, etc. Use this in app code.
- **Namespaced (disambiguation)**: `<x-pn::button>` — use only when a consumer app has its own `<x-button>` that conflicts.
- **Never use the old `<x-pinion-ui::xxx>` prefix** — it was renamed to `pn::` in v0.2.1.

### Nested parent + children (v0.4.0+)

Two components compose as a parent with nested children rather than driven by a single array prop:

| Parent | Child | What the child carries |
|---|---|---|
| `<x-tabs>` | `<x-tab name label :icon>{{ slot }}</x-tab>` | One tab button + one panel. |
| `<x-accordion>` | `<x-accordion-item title :name>{{ slot }}</x-accordion-item>` | One header + one disclosure region. |

```blade
<x-tabs variant="boxed">
    <x-tab name="overview" label="Overview"><p>…</p></x-tab>
    <x-tab name="specs"    label="Specs"><p>…</p></x-tab>
</x-tabs>
```

How it works under the hood (so the pattern doesn't surprise you):

- **Shared props via `@aware`**: parent props (`variant`, `size`, `multiple`) flow into children through Blade `@aware`. Don't declare them again on the child call site — they're inherited.
- **Shared state via Alpine scope**: the parent owns the only `x-data` (`activeTab` / `open`); children read and write it via Alpine's normal scope chain. Don't add a new `x-data` on a child.
- **Composer is called from both Blades**: parent and child each call the same `Composer::compose()` so the class strings line up. This stays consistent with the rest of the architecture rules below.

Per-component docs cover the full prop tables and slot contracts: [`reference/components/tabs.md`](./reference/components/tabs.md), [`reference/components/accordion.md`](./reference/components/accordion.md). The previous array-driven shape (`:tabs="[…]"` / `:items="[…]"`) was removed in v0.4.0 — see [`SEMVER.md`](./SEMVER.md).

## Architecture rules (do not violate)

1. **Compose pattern**: For most components, class strings live in `src/Compose/{Name}Composer.php`, not the Blade. Each composer has a static `compose(array $props): array` that returns a flat dict of class strings. The Blade reads `$c['root']`, `$c['title']`, etc. — it is render-only.
   - Components **without** a composer (classes embedded in the Blade): `button`, `alert`, `card`, `badge`, `avatar`, `menu-item`, `section.hero`, `theme-switcher`. These predate the pattern.
2. **Composer returns class strings only — no markup, no array values.** If you need conditional markup, branch in the Blade based on a prop.
3. **Fixture tests**: Each composer has `tests/fixtures/compose/{name}.json` covering its variant matrix. Run with `php tests/Compose/run.php`. Comparison is subset (only listed `expected` keys are checked). Add cases when you add props.
4. **Backwards compatibility**: Never rename props or silently change their defaults. New props are opt-in (default preserves previous behaviour).

## Three style layers (orthogonal)

| Layer | Where it lives | Examples |
|---|---|---|
| **Theme** (color palette) | `<html data-theme="...">` | `pinion`, `pinion-dark`, `monokai`, `payments-dark` (37 original light/dark pairs — see the theme lineup section below; daisyUI's built-in themes do **not** exist in the build) |
| **Tune** (shape / space / font) | `<html data-tune="...">` | `default`, `minimal`, `tech`, `editorial`, `soft` (11 presets) |
| **Component** (variant / size / state) | Blade props | `color="primary"`, `size="lg"`, `dismissible` |

Themes and Tunes mix freely. Both are activated by the `pinion-ui.css` preset (imported into `app.css` by `ui:install`), which bundles the `data-tune` token system from `tune.css`.

### Class vocabulary — what you actually write


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sparrowhawk-labs/pinion-ui](https://github.com/sparrowhawk-labs/pinion-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
