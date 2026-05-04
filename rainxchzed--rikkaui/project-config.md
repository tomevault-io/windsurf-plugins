---
trigger: always_on
description: Provides LocalContentColor to children. Content lambda: @Composable () -> Unit
---

# RikkaUi — Project Context

## What This Is

**RikkaUi** is a shadcn/ui-inspired component library + design system for Compose Multiplatform.
Name means "snowflake" (六花) or "composing elements into harmony" (立花).
Tagline: "Share UI via Compose Multiplatform UI framework"

**The gap:** No one combines (1) styled components (2) copy-paste ownership (3) registry system (4) CMP support. We fill it.

## Architecture

### Modules

- `:foundation` — Theme tokens and design system foundation. Multiplatform: Android, Desktop (JVM), iOS, WasmJs.
- `:components` — The component library (40+ components). **No Material3 dependency.** Built on `compose.foundation` only. Multiplatform: Android, Desktop (JVM), iOS, WasmJs. Depends on `:foundation`.
- `:composeApp` — The showcase website. WasmJs (browser). Depends on `:components`.
- `:feature:docs` — Documentation pages for all components. Depends on `:components`.
- `:feature:creator` — Design system creator/configurator page.

### Package Structure

```
foundation/src/commonMain/kotlin/zed/rainxch/rikkaui/foundation/
    RikkaColors.kt       — 31 semantic color tokens (@Immutable, staticCompositionLocalOf)
                            + LocalContentColor for implicit foreground propagation
                            + on* naming pattern for content colors (onBackground, onSurface, onPrimary, etc.)
                            + Hover/press tokens (primaryHover, primaryPressed, etc.) default to Color.Unspecified
                            + Tinted container tokens (primaryTinted, destructiveTinted)
    ColorScheme.kt        — 5 base palettes (Zinc/Slate/Stone/Gray/Neutral) x Light/Dark = 10 schemes
                            + 7 accent colors (Red/Rose/Orange/Green/Blue/Yellow/Violet) x Light/Dark
                            + withAccent() extension + RikkaAccentColor data class (onPrimary, primaryHover, primaryPressed)
                            + RikkaPalette enum + RikkaAccentPreset enum with resolve()/applyTo()
                            + All palettes include hover/press values following Tailwind color scale
    RikkaTypography.kt    — 9-level type scale (h1-h4, p, lead, large, small, muted)
    RikkaSpacing.kt       — 7-level spacing scale (xs=4dp through xxxl=48dp, 4dp base grid)
    RikkaShapes.kt        — 5-level shape scale (sm/md/lg/xl/full) from base radius
    RikkaMotion.kt        — Animation token system (springs, tweens, durations, press scales)
    RikkaElevation.kt     — Shadow elevation scale (none/low/medium/high) for cards, dialogs, sheets, FABs
    RikkaStyle.kt         — RikkaStyle data class + RikkaStylePreset enum (Default/Nova/Vega/Aurora/Nebula)
    RikkaFontFamily.kt    — Font wrapper with rememberRikkaFontFamily() composable
    RikkaTheme.kt         — 4 RikkaTheme overloads + RikkaTheme object (see Theme System section)
    modifier/
        KeyboardScrollable.kt — Keyboard scrolling modifier for scrollable containers (Arrow/Space/Page/Home/End)
                                 + @Composable 1-param overload (zero boilerplate, just pass ScrollState)
                                 + ScrollFocusMode enum (RequestFocus/Hover/Click) for focus acquisition strategy
        MinTouchTarget.kt     — WCAG minimum touch target enforcement (48dp default)
                                 + LocalMinTouchTarget CompositionLocal (override to 0.dp in dense contexts like popups)
        FocusRing.kt          — Focus ring border using theme ring color (shadcn focus-visible:ring-2 equivalent)
        StaggeredEnter.kt     — Stagger delay strategy for list enter animations (Fast/Default/Slow)

components/src/commonMain/kotlin/zed/rainxch/rikkaui/components/ui/
    text/Text.kt          — BasicText wrapper with TextVariant enum, heading accessibility
                             Color resolution: explicit → LocalContentColor → variantColor()
    button/Button.kt      — 6 variants (Default/Secondary/Destructive/Outline/Ghost/Link), 4 sizes (Default/Sm/Lg/Icon), 3 animations
                             Provides LocalContentColor to children. Content lambda: @Composable () -> Unit
    button/IconButton.kt  — Convenience wrapper over Button with ButtonSize.Icon. 3 sizes (Sm/Default/Lg). Defaults to Ghost variant.
    card/Card.kt          — 3 variants (Default/Ghost/Elevated) + CardHeader/CardContent/CardFooter
                             Elevated uses subtle border (alpha 0.5) for dark mode visibility
    badge/Badge.kt        — 4 variants (Default/Secondary/Destructive/Outline), text + content overloads
    separator/Separator.kt — Horizontal/Vertical, decorative (clearAndSetSemantics)
    input/Input.kt        — BasicTextField wrapper, animated focus border, placeholder, accessibility
    toggle/Toggle.kt      — Spring-animated thumb, 2 sizes, thumb uses onPrimary when checked
    checkbox/Checkbox.kt   — Animated checkmark, label renders visually (not just a11y)
    radio/RadioButton.kt   — Radio selection control
    textarea/Textarea.kt   — Multi-line text input
    label/Label.kt         — Form label with disabled state
    skeleton/Skeleton.kt   — Pulsing loading placeholder
    spinner/Spinner.kt     — Rotating loading indicator (3 sizes), defaults to LocalContentColor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rainxchzed/RikkaUi](https://github.com/rainxchzed/RikkaUi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
