---
trigger: always_on
description: These rules apply to every coding agent working in this repository. They are the
---

# ChatYX Agent Instructions

These rules apply to every coding agent working in this repository. They are the
short form of the engineering contract; `documents/ARCHITECTURE.md` and
`documents/DESIGN.md` hold the detail.

## Required reading

Before non-trivial work, read what the change touches:

- `documents/ARCHITECTURE.md` — layers, ownership, the setup feature, icons, tokens
- `documents/DESIGN.md` — the token contract and the overlay's visual model
- relevant tests for the area being changed

Repository-wide refactors additionally require:

- `documents/REFACTORING.md` — the full engineering rules and refactoring workflow
- `documents/REFACTORING_FOLLOWUP.md` — the current completion supplement

Do not start a repository-wide cleanup after reading one or two files.

## Architecture direction

```text
routes -> features -> services/config/utils
                    -> platform adapters
```

- `routes/` bind Solid signals and JSX to a feature's public API. A route owns
  visibility and composition, not infrastructure, network clients or timers.
- `features/*/application` coordinate use cases and own lifecycle.
- `features/*/model` hold pure state transformations and calculations.
- `services/` implement network, browser, provider, storage and platform
  capabilities.
- `config/` holds schemas, normalization, parsing, serialization and defaults.
- `utils/` holds small genuinely reusable helpers, never domain logic.

A service must not import a feature, a route or a component. A feature must not
import a component: shared contracts live in `features/*/model`, `config/` or the
`services/` layer, and components import them, never the reverse. The one
recorded exception is that a `config/` module may depend on a dependency-free
`services/` leaf (`config/setupTemplates` on `services/storage/setupStorage`).

Do not add a service locator or a DI container. Pass dependencies through
constructors, factories or parameters.

## Composition roots and lifecycle

Every long-lived resource — socket, timer, interval, subscription, observer,
listener, mutable cache, reconnect loop, runtime — has exactly one owner, and
that owner releases it.

- `createChatOverlayApplication` is the chat route's composition root: one live
  or preview runtime plus one predictions controller.
- Construction must not create an owned resource that `destroy()` refuses to
  release. `destroy()` is safe before `start()`, safe when called twice, and
  safe while `start()` is still awaiting.
- Teardown is an ownership boundary, not a reset call: asynchronous work started
  under a runtime must not repopulate shared state after that runtime is
  destroyed. Singleton services guard their commits with a generation they
  capture when the work starts.
- Preview and live runtime own different things in the same document. Reset a
  process-wide singleton only from the owner that installed it.
- Browser-global events are for real browser integration only, never as an
  internal event bus.

## UI primitives

`src/components/ui/` holds generic primitives with no ChatYX domain knowledge —
no Twitch, YouTube, Kick, OBS, TTS, RTE, setup-section or chat-config concepts.

```text
components/ui        generic primitive
components/<feature> feature presentation
routes/*             page composition
```

Check for an existing primitive before writing a control. Do not push a bespoke
control through a generic primitive when that changes its design; extract only
genuine duplication, and put feature wrappers in `components/<feature>/`.

## Icons

- Generic UI icons: `@hugeicons/core-free-icons`, rendered through
  `src/components/ui/icon.tsx`, which is a thin wrapper over the vendor's
  `HugeiconsIcon` from `@hugeicons/solid-js`. The wrapper owns `currentColor`,
  the `1em` default size and decorative-by-default accessibility; the vendor
  owns SVG rendering; the icon payload owns its stroke width.
- Import icons by subpath (`@hugeicons/core-free-icons/TextIcon`). Never
  `import * as Icons`, never a runtime string lookup, never an icon font.
- Brand glyphs (Twitch, YouTube, Kick, GitHub, 7TV) have one owner:
  `src/components/brand/PlatformGlyph.tsx`. Do not replace them with generic
  icons and do not duplicate them as a public SVG.
- Bespoke graphics — sparklines, event, reply and role artwork, custom
  visualizations — stay custom. Consistency means one system per
  responsibility, not one renderer for every drawing.
- Decorative icons stay hidden from assistive technology; icon-only controls
  carry an accessible name.

## Styling

Tailwind, CVA, `cn()` and Kobalte are the current stack. Do not add another CSS
framework, and do not remove Tailwind to prepare for a future one.

- Static CSS owns rule shape. The runtime publishes values as CSS custom
  properties: `getOverlayStyleVariables()` → `OverlayStyleManager` → document
  root → `chat.css`. Do not go back to one generated stylesheet per preset.
- Generated CSS is valid only where the rule shape is provider data: 7TV paint
  and emote modifiers. Ask whether stable CSS plus a custom property expresses
  the behaviour before generating CSS.
- A reusable design decision becomes a token; a value that changes while the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruinateam/ChatYX](https://github.com/ruinateam/ChatYX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
