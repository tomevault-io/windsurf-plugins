---
trigger: always_on
description: These rules are mandatory for every agent working in this repo. The identical set
---

# Repository Guidelines

## Rules (always apply)

These rules are mandatory for every agent working in this repo. The identical set
lives in `CLAUDE.md` — keep the two copies byte-for-byte in sync.

1. **Keep CLAUDE.md and AGENTS.md synchronized.** Whenever `CLAUDE.md` (Claude's
   memory) is changed, mirror the same change into `AGENTS.md`, and vice versa, so
   all agents read one shared source of truth. Never let the two drift.
2. **Design tokens are mandatory.** When adapting any Zag.js component into Manti
   UI or authoring a new Manti UI component, every visual value — color, spacing,
   radius, typography, motion, elevation, z-index — must come from the design
   tokens (`@manti-ui/tokens` / token-backed `--manti-*` and `--variant-*` CSS
   variables). Never hard-code raw values (hex colors, px sizes, magic numbers);
   if a token is missing, add it to `@manti-ui/tokens` first, then consume it.
   `@manti-ui/tokens` is the single source of truth: its primitive ramps and
   scale values are **generated** into the `--manti-*` custom properties in
   `packages/styles/src/tokens.css` (the `@tokens:generated` region) by
   `pnpm gen:tokens`. Never hand-edit that region; after changing the contract,
   regenerate it — the styles build fails if it is stale. The theme-aware roles
   (`light-dark()` surfaces/text/elevation/panel) and the `--variant-*`
   vocabulary below the region stay hand-authored. Tokens form three tiers:
   primitive ramps → semantic roles/variants → **component tokens**
   (`--manti-{component}-{property}`, public and semver-stable, each defaulting
   to a semantic token). When a component needs an _independent_ structural value
   (radius, padding, sizing, gap, typography), expose it as a component token
   rather than a bare literal or a private knob; keep only _derived_ `calc()`
   values as private `--_*`. Register every component token in the
   `componentTokens` map of `@manti-ui/tokens`.
3. **Color-scale roles and interaction progression are mandatory.** Primitive
   ramps are ordered `1`–`12`, but components must consume semantic roles or the
   `--variant-*` vocabulary rather than picking primitive stops directly. Use
   `1`–`2` for canvas/subtle surfaces; `3` for component rest, `4` for hover and
   keyboard highlight, `5` for pressed/selected/checked; `6` for quiet chrome,
   `7` for interactive borders and low-emphasis controls, `8` for focus rings
   and strong interactive chrome; `9` for solid fills, `10` for solid hover;
   `11` for supporting text and `12` for high-contrast text. Input-like controls
   (including Input, Textarea, Select, Combobox, NumberInput, PinInput,
   TagsInput, Editable, DatePicker, TimePicker, ColorPicker, and Clipboard) use
   `--manti-border` at rest, strengthen to the neutral
   `--manti-border-strong` on hover, and use `--variant-ring` while active,
   focused, or open. The primary color must begin at active/focus/open for this
   control family; never use a variant color for its resting or hover border.
   Select is the deliberate open-state exception: once its popup is connected,
   both trigger and popup borders become transparent.
   Semantic mappings must be theme-aware: interaction strength increases in the
   direction that gains contrast in each theme.
   Filled and selected surfaces follow rest `3` → hover `4` → active `5`; solid
   controls follow `9` → `10`. Define these mappings centrally in
   `packages/styles/src/tokens.css`, consume only semantic/variant roles in
   component CSS, and verify changes with `pnpm check:color-scale`,
   `pnpm --filter @manti-ui/styles check:contrast`, and the styles build.
4. **Never use box shadows.** `box-shadow` is prohibited everywhere in Manti UI,
   including components, utilities, documentation surfaces, examples, motion
   states, and focus treatments. Do not add shadow tokens or shadow-based
   elevation. Communicate hierarchy with surface contrast, borders, spacing,
   and layering instead; use an outline only when it conveys a real boundary,
   focus state, or functional mask rather than simulated elevation.
5. **Match the user's language.** Always reply in the same language the user wrote
   their prompt in (e.g. Turkish prompt → Turkish answer). This applies to chat
   responses only; code, identifiers, comments, and docs stay in English.

## Project Structure & Module Organization

This is a `pnpm` workspace for a framework-agnostic design system powered by
Zag.js behavior machines.

- `packages/tokens/`: shared design-token contract.
- `packages/styles/`: shared CSS and state selectors.
- `packages/folds/`: framework-agnostic Zag.js behavior.
- `packages/react/`: React renderer and Storybook stories.
- `.storybook/`: Storybook (react-vite) configuration — the single dev surface and visual gallery.
- `design/logo-explorations/`: non-production brand exploration assets.
- `docs/`: architecture and product vision.

Keep component stories beside their implementation when practical. Export
public React APIs through `packages/react/src/index.ts`.

## Build, Test, and Development Commands

Use Node `>=22.12.0` and `pnpm 10`.

```bash
pnpm install          # Install all workspace dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manti-ui/ui](https://github.com/manti-ui/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
