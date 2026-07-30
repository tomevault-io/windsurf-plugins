---
trigger: always_on
description: Read this before touching `mobile/`. It is the native analog of the web app's
---

# Lyftr Mobile — Conventions

Read this before touching `mobile/`. It is the native analog of the web app's
conventions: same design language, same tokens, same component names — expressed in
React Native / Expo idioms. When in doubt, open `web/src/components/ui/` and mirror it.

## 1. Reuse boundary — logic is shared, UI is per-platform

- **Logic lives in `@lyftr/shared`** and is *imported, never duplicated*: types, the
  API client, Zustand stores, and pure utils (`displayWeight`, `normalizeServerUrl`,
  `testServerConnection`, ...). If you're about to write a type, a fetch, or a
  unit-conversion in `mobile/`, stop — it belongs in `packages/shared`.
- **UI is authored per-platform**, but mirror the web's component names and prop
  shapes wherever an analog exists (`PageHeader`, `SectionHeader`, `SegmentedControl`,
  `EmptyState`, `IconButton`, `StepperTile` + `NumberField`, ...), adapted to RN
  idioms: `onPress` not `onClick`, `TextInputProps` not input attributes,
  `accessibilityLabel` not `aria-label`. The two apps must stay legible to each other.
- **Don't re-roll primitives.** Read `src/components/ui/` first; if a screen needs a
  visual pattern twice, extract it there.

## 2. Styling rule (the one rule to remember)

**Default: NativeWind `className`**, using the shared token vocabulary — identical to
the web's: `bg-surface-{base,raised,overlay,border,muted}`, `text-tx-{primary,
secondary,muted,inverse}`, `bg-brand-500`, `text-error-400`, `border-surface-border`,
opacity modifiers like `bg-brand-500/10`. These resolve from CSS variables
(`global.css`) and flip with the theme automatically.

**Inline `style` + `useTheme().colors|brand` is required only where className can't
reach.** The narrow, exhaustive exceptions:

1. **Animation-driven values** — anything fed by Reanimated / `Animated`
   (`useAnimatedStyle`, interpolations). See `Field.tsx`, `authui.tsx`.
2. **Gradients** — `expo-linear-gradient` takes a `colors` prop, use
   `useTheme().brand.gradient`.
3. **SVG / icon props** — `lucide-react-native` `color`, `react-native-svg`
   `stroke`/`fill` are component props, not styles.
4. **Values NativeWind can't express** — `fontVariant: ['tabular-nums']`, measured
   layout math, `shadowColor` glows, platform-conditional numbers.

**Litmus test:** *if the value changes every frame, or is passed as a component prop
rather than a style, theme it inline via `useTheme()`; everything else is
`className`.*

The auth screens (`authui.tsx`, `AuthScaffold.tsx`, `(auth)/*`) are grandfathered
100%-inline — they are working, Fabric-tuned code. Don't churn them to match this
rule; do follow the rule in everything new.

## 3. Theming

- **Always go through tokens** — `className` tokens or `useTheme()` (`colors`,
  `brand`, `accent`). Never hard-code hex. `accent` exists because raw brand cyan is
  illegible on light surfaces: it resolves to `cyanEdge` (light) / `cyanLight` (dark).
- Soft red (`errorSoft`/`error-400`) reads on dark but washes out on light — pick via
  `isDark` (see `AuthError`, `IconButton`'s danger variant).
- **Known debt** (hard-coded hex to migrate to tokens when next touched — don't fix
  drive-by): `AuthScaffold` gradient `#00b8d9/#8b5cf6` → `brand.gradient`;
  `WeightChart` stroke `#00b8d9` → `brand.cyan`; `authui` `LABEL_COLOR #94a3b8`;
  `weight.tsx` `Trash2 #f87171`; `RefreshControl tintColor #00b8d9` (dashboard) and
  the `_layout.tsx` ActivityIndicator.

## 4. Typography — use the primitives, or the fonts don't render

The brand fonts (Outfit for display, Plus Jakarta Sans for body) are loaded
**per-weight** in `app/_layout.tsx` — each weight is its own `fontFamily`
(`Outfit_700Bold`, `PlusJakartaSans_600SemiBold`, ...). Tailwind's `font-bold` sets
`fontWeight`, which does **not** select those families: bare `font-bold` /
`font-semibold` text silently renders in the *system* font.

Therefore:

- **All text goes through `src/components/ui/Typography.tsx`** — `AppText` (variant +
  color props) or its wrappers `Heading`, `Body`, `Label`, `Numeric`, plus the legacy
  `H1` / `Muted`.
- **Bare `font-bold`/`font-semibold` on brand text is forbidden.** Inside `ui/`
  primitives, use the family classes (`font-display`, `font-display-heavy`,
  `font-sans`, `font-sans-semibold`, `font-sans-bold`, `font-sans-heavy`) from
  `tailwind.config.js`.
- Scale: `display` 28 / `title` 24 (page H1) / `heading` 18 (Outfit); `subheading` 14 /
  `body` 15 / `bodySemibold` 15 / `caption` 12 / `label` 11-uppercase (Jakarta);
  `numeric` 30 tabular (Outfit 800) for metric values.
- The web uses JetBrains Mono for numerics; mobile approximates with Outfit +
  `tabular-nums` rather than loading a third family (debt: revisit if numbers ever
  need true mono).
- Don't stack a size class on a sized variant (`<AppText variant="body"
  className="text-xs">`) — two `fontSize` classes leave the winner to stylesheet
  order. Pick the right variant; `Muted` intentionally has no size so its callers'
  `text-xs` keeps working.

## 5. Component authoring

- One component per file in `src/components/ui/`, **named exports**, re-exported from
  the barrel `index.ts` (screens import the folder: `../../src/components/ui`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cawlumm/lyftr](https://github.com/Cawlumm/lyftr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
