---
trigger: always_on
description: Scope: this file applies to everything under `ui/`. AI coding agents (Claude Code, Cursor, etc.) load this file automatically when working in this directory; humans should treat it as the source of truth for frontend conventions in Kestra.
---

# UI Design System Guidelines

Scope: this file applies to everything under `ui/`. AI coding agents (Claude Code, Cursor, etc.) load this file automatically when working in this directory; humans should treat it as the source of truth for frontend conventions in Kestra.

The Kestra design system lives at [ui/packages/design-system/](packages/design-system/) and is the **single source of truth** for every visual element of the product — colors, fonts, spacing, buttons, forms, dialogs, tables, charts, and so on. Anything rendered to a user must come from it.

## What this is, in plain terms

Think of the design system as the product's **visual vocabulary**:

- A short list of agreed-upon **colors**, **fonts**, and **spacings** (called *design tokens*).
- A library of pre-built **components** (`KsButton`, `KsTable`, `KsDialog`, …) that already use those tokens.
- A guarantee that anything built from these pieces will look right in **light mode and dark mode**, follow accessibility rules, and stay visually consistent with the rest of Kestra.

If a screen feels "off-brand," looks broken in dark mode, or every page styles the same control differently, it's almost always because someone bypassed the design system. The rules below exist to prevent that.

Under the hood, the design system wraps Element Plus under the `kel` namespace and globally registers every component with a `Ks*` prefix. You should almost never `import` from `element-plus` directly in `ui/src/`.

> **Note on `@kestra-io/ui-libs`:** The codebase may still contain imports from `@kestra-io/ui-libs`, the previous shared component library. That repository is sunsetting — all components have been migrated here into `ui/packages/`. Do not add new imports from `@kestra-io/ui-libs`; use `Ks*` components from the design system instead.

## Golden rules (non-negotiable)

These rules are what keep the UI maintainable as it grows. Treat any deviation as a bug.

1. **Use a `Ks*` component if one exists.** Check the tables below before writing anything custom or importing from `element-plus`. New screens that mix `<el-button>` and `<KsButton>` are a regression.
2. **Colors come from `--ks-*` tokens. Always.** No hex codes, no `rgb(...)`, no Element Plus tokens (`--el-*`), no Bootstrap variables, no SCSS color variables in component code. If the token you need does not exist, talk to design and add it to `ks-theme-light.scss` / `ks-theme-dark.scss` / `ks-theme-dark-2.scss` — do not pick a one-off color.
3. **Typography comes from `KsText` or typography tokens.** Use `<KsText>` (with `size`, `type`, `tag`, `truncated`, `lineClamp`) for body copy. For headings or one-off needs, use the `$font-family-*` and `$font-size-*` SCSS variables only inside the design-system package — feature code should not redefine them.
4. **No `:deep()` selectors.** Reaching into a child component's internals breaks encapsulation and silently shatters when the design system is upgraded. If you need to style something inside a `Ks*` component, add a prop, a slot, or a CSS variable to the component upstream.
5. **No SCSS variables (`$...`) in feature components.** Use `var(--ks-*)` CSS custom properties inside `<style>` blocks. SCSS variables don't react to dark mode, can't be overridden at runtime, and bind your component to a specific theme. SCSS variables are only acceptable inside `ui/packages/design-system/` itself, in mixins, or for math at build time.
6. **No magic numbers for theme values.** Spacing, radii, font sizes, and shadows must reference tokens or design-system SCSS variables — never `padding: 13px`, never `border-radius: 6px`. For spacing (`padding`/`margin`/`gap`), reach for the `--ks-spacing-*` scale first (`--ks-spacing-1` = 0.25rem, `-2` = 0.5rem, `-3` = 0.75rem, `-4` = 1rem, `-5` = 1.5rem, `-6` = 2rem, `-7` = 2.5rem, `-8` = 3rem, `-10` = 4rem, `-12` = 5rem, `-16` = 6rem; declared in [`ks-tokens.scss`](packages/design-system/src/assets/styles/ks-tokens.scss)). Only fall back to a raw `rem` value when no token fits — never a hardcoded `px` value (`margin: 0 24px` → `margin: 0 var(--ks-spacing-5)`).
7. **Never override Element Plus classes directly.** Don't write `.el-button { ... }` in feature code. If a `Ks*` component is missing a behavior, extend the component in the design system instead of patching CSS at the call site.
8. **Don't fork — extend.** If a `Ks*` component is *almost* what you need, add a prop or a slot to the component in `ui/packages/design-system/`. Copy-pasting the component into your feature folder is forbidden.
9. **Every new `Ks*` component needs a Storybook story and a unit test.** Stories double as living documentation for design and product reviewers.
10. **i18n keys live with the design system component**, not inside feature code, when they belong to the component (e.g. `KsEmpty`, `KsDurationPicker`). Register them via `registerDesignSystemI18n`.

## Best practices for keeping the design system healthy

A design system rots fast if it's treated as a one-time deliverable. Apply these rules every time you touch UI code or review a UI PR.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kestra-io/kestra](https://github.com/kestra-io/kestra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
