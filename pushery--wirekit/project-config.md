---
trigger: always_on
description: WireKit component library conventions for AI tooling
---


# WireKit — AI Authoring Rules

WireKit is a Laravel Livewire component library. When generating Blade
templates, CSS, or PHP that uses WireKit, follow these rules.

## Component invocation

WireKit components use Laravel's anonymous-component namespace syntax
with a double colon:

```blade
<x-wirekit::button intent="primary">Save</x-wirekit::button>
<x-wirekit::input name="email" wire:model="email" />
<x-wirekit::card>
    {{-- Card is a FRAME (radius/border/bg, NO padding). Content goes in card.body. --}}
    <x-wirekit::card.body>
        <x-wirekit::heading>Title</x-wirekit::heading>
        <x-wirekit::text>Body copy</x-wirekit::text>
    </x-wirekit::card.body>
</x-wirekit::card>
```

NEVER use a single colon (`<x-wirekit:button>`) — that syntax does not
resolve to the package's anonymous-component path.

## Variant system

Most interactive components support a unified Intent × Surface variant
system. Acceptable values:

- **Intent:** `primary`, `neutral`, `success`, `warning`, `danger`, `info`
  (the shared set, from `VariantResolver::INTENTS`). `secondary` is NOT an
  intent — it is a legacy `variant` value that maps to `intent="neutral"
  surface="filled"`. `badge` additionally accepts `intent="accent"`.
- **Surface:** `filled` (default), `soft`, `outline`, `ghost`, `link`
  (from `VariantResolver::SURFACES`). `badge` uses a different surface set:
  `soft` (default), `solid`, `outline` — `solid` is badge-only, not a button
  surface.

The color-role prop is `intent` (button / badge / progress); `surface` selects the
visual treatment. When in doubt, prefer `intent="primary"` over hand-coding colors.

NEVER apply Tailwind color utilities to recolor a component:

```blade
{{-- ❌ WRONG --}}
<x-wirekit::button class="bg-blue-500 text-white">…</x-wirekit::button>

{{-- ✅ RIGHT --}}
<x-wirekit::button intent="primary">…</x-wirekit::button>
```

## Design tokens

WireKit ships its design tokens under the `--*-wk-*` namespace. They
auto-switch in dark mode via the `.dark` class. NEVER hand-write color
values; reach for the token. NEVER use Tailwind palette classes
(`zinc-*`, `gray-*`, `slate-*`) inside WireKit components or their
overrides.

Common tokens:

- `--color-wk-bg`, `--color-wk-bg-muted`, `--color-wk-bg-input`
- `--color-wk-text`, `--color-wk-text-muted`, `--color-wk-text-subtle`
- `--color-wk-border`, `--color-wk-border-subtle`, `--color-wk-border-hover`, `--color-wk-border-strong`, `--color-wk-border-strong-hover` (form controls MUST use the `strong` pair — the communicating border, WCAG 1.4.11, ≥3:1), `--color-wk-border-error`, `--color-wk-border-success`
- `--color-wk-accent` (fill), `--color-wk-accent-text` (accent-toned text on bg — alias of `--color-wk-accent-content`), `--color-wk-accent-fg` (text on an accent fill)
- `--space-wk-xs|sm|md|lg|xl|2xl` (+ `--space-wk-section-sm|md|lg`)
- `--gap-wk-xs|sm|md|lg|xl|2xl` (the full `xs`…`2xl` internal layout-gap ladder). Note: the `<x-wirekit::stack|row|grid>` `gap` prop maps to `--space-wk-*`, not these — the `--gap-wk-*` tokens are used internally by composed components (`brand`, `header`, `cta`, `hero`, `navbar`, …).
- `--padding-wk-x-xs|sm|md|lg|xl` and `--padding-wk-y-xs|sm|md|lg|xl` (axis-split — no axis-less shorthand)
- `--radius-wk-sm|md|lg|xl|full`
- `--shadow-wk-sm|md|lg`
- `--text-wk-2xs|xs|sm|md|lg|xl|2xl|3xl|4xl|5xl`
- `--transition-wk-duration`, `--transition-wk-easing`

NEVER use the `dark:` Tailwind prefix on a WireKit component or its
content — tokens auto-switch via the parent `.dark` class.

## Icon usage

```blade
<x-wirekit::icon name="check" size="sm" />
<x-wirekit::icon name="x-mark" size="md" aria-label="Close" />
```

Acceptable `size` values: `xs` (12px), `sm` (16px), `md` (20px),
`lg` (24px), `xl` (32px). NEVER use `class="h-N w-N"` on
`<x-wirekit::icon>` — the `size` prop is the canonical way.

To place an icon inside a `<x-wirekit::button>` (or `link`), use the
`iconLeft` / `iconRight` slots — they position and space the icon
correctly and shrink-proof it:

```blade
<x-wirekit::button intent="primary">
    <x-slot:iconLeft><x-wirekit::icon name="plus" size="sm" /></x-slot:iconLeft>
    New item
</x-wirekit::button>
```

Stackable presets — base preset (`heroicons`) plus optional
`heroicons-app` and `heroicons-marketing` extensions add app- and
marketing-specific aliases. Configure in `config/wirekit.php`:

```php
'icons' => ['presets' => ['heroicons', 'heroicons-marketing']],
```

## Layout primitives

Reach for these instead of hand-rolled flex/grid:

- `<x-wirekit::container>` — max-width wrapper with responsive padding
- `<x-wirekit::stack gap="md">` — vertical flex column (the prop is `gap`, NOT `space`)
- `<x-wirekit::row gap="sm">` — horizontal flex row (the prop is `gap`, NOT `space`)
- `<x-wirekit::grid cols="3" gap="md">` — responsive CSS grid
- `<x-wirekit::section>` — semantic page section with vertical padding
- `<x-wirekit::center>` — centers a single child both axes
- `<x-wirekit::aspect-ratio ratio="16/9">` — locked aspect ratio
- `<x-wirekit::divider>`, `<x-wirekit::spacer>` (the spacer is a flex-grow gap — it has no `size` prop)

Components carry NO outer margins — compose rhythm with `stack`/`row`/`grid`/
`section` + their `gap` prop. Do NOT hand-roll `space-y-*` / `mb-*` utilities.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pushery/wirekit](https://github.com/pushery/wirekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
