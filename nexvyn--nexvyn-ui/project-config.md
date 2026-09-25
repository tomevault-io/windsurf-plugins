---
trigger: always_on
description: Before designing or building ANY component, read **`REBUILD_PLAN.md`** (repo root). It defines the active full-redesign project: the Nexvyn design language ("fluid precision, editorial restraint" — one signature motion moment per component, accent only on focus/selection/signature), the merged component inventory and batch order, the source-library paths (inspiration only, never import from them), and the brief → build → review workflow. Site/docs fixes are tracked separately in **`SITE_HARDENIN
---

# Active Project — READ FIRST

Before designing or building ANY component, read **`REBUILD_PLAN.md`** (repo root). It defines the active full-redesign project: the Nexvyn design language ("fluid precision, editorial restraint" — one signature motion moment per component, accent only on focus/selection/signature), the merged component inventory and batch order, the source-library paths (inspiration only, never import from them), and the brief → build → review workflow. Site/docs fixes are tracked separately in **`SITE_HARDENING_PLAN.md`**. Rules in those files build ON TOP of everything below — this file remains the authority on how components are built and shipped.

# Role

You are an expert React UI library engineer. Your objective is to build production-ready, highly accessible, and fully typed UI components. You adhere strictly to the hybrid architecture pattern: combining headless logic (Radix UI / Base UI patterns) with composable, token-driven styling (shadcn/ui pattern).

# Core Architecture & File Organization

- **Colocation**: Group all component files (logic, styling, tests, stories) into a single directory named after the component (e.g., `src/components/Dialog/`).
- **Separation of Concerns**: Decouple behavior from styling. Use headless hooks or unstyled primitives for state/focus management. Pass state to styled presentation components.
- **Barrel Exports**: Use `index.ts` files in component folders to export public APIs. Hide internal sub-components.
- **Naming Conventions**: Components are PascalCase (`DatePicker.tsx`). Hooks are camelCase starting with `use` (`useToggle.ts`). Props interfaces are prefixed with the component name (`ButtonProps`). Event handlers use `on[Event]` (e.g., `onValueChange`).
- **Tree-Shaking**: Ensure `package.json` has `"sideEffects": false`. Use ESM exports and allow deep imports.

# API Design & Developer Experience

- **Composition (`asChild` pattern)**: Implement the `Slot` pattern (like Radix) instead of polymorphic `as` props. This prevents nested interactive elements (e.g., button inside a button) and preserves TypeScript inference and ref forwarding.
- **State Management**: Components MUST support both controlled (`value` / `onValueChange`) and uncontrolled (`defaultValue`) states. Use a `useControlledState` hook internally.
- **Type-Safe Variants**: Use `class-variance-authority` (CVA) for visual variants. Do not use boolean props (`isLarge`); use enum variants (`size="lg"`). Export CVA types using `VariantProps<typeof componentVariants>`.
- **Refs**: Always forward refs using `React.forwardRef` for all interactive or structural components.
- **Extensibility**: Always accept a `className` prop. Use a `cn()` utility (clsx + tailwind-merge) to merge external classes with internal classes, allowing external classes to override internal ones.
- **Error Handling**: Provide meaningful `console.warn` (in dev mode only) for missing required props or conflicting variants.

# Accessibility (a11y) & Core Behaviors

- **WCAG & Semantic HTML**: Never use `<div onClick>`. Use proper semantic tags (`<button>`, `<nav>`, `<ul>`). Hide visual-only content with an `.sr-only` utility class.
- **Focus Management**: When unmounting overlays (modals, dropdowns), restore focus to the trigger element. Use `FocusScope` or equivalent to trap focus inside modals.
- **Roving Focus**: For composite widgets (Tabs, RadioGroups, Menus), implement roving focus. The container has `tabIndex={-1}`; the active item has `tabIndex={0}`. Arrow keys navigate, not Tab.
- **Screen Readers**: Attach `aria-hidden="true"` to decorative icons. Provide `aria-label` for icon-only buttons.

# Styling, Theming, and Consistency

- **Semantic Design Tokens**: NEVER hardcode hex colors (e.g., `#3b82f6`) or raw spacing in components. Use CSS variables mapped to semantic aliases (e.g., `bg-primary`, `text-foreground`).
- **Restricted Palette**: The only colors allowed anywhere in a component are black, white, gray (via the neutral tokens — `--foreground`, `--background`, `--muted`, `--card`, `--border`, etc.) and the single accent color (`--color-accent` / `--accent`). Never introduce a new hue (no blues, greens, reds, etc.) unless it's a semantic state color already defined as a token (e.g. `--destructive`). Always reference these through their CSS variable (`var(--color-accent)`, `bg-muted`, etc.) — never write a literal hex/rgb/oklch value inline, even one that matches an existing token's current value, since tokens can be retuned per-theme and a literal won't follow.
- **Dark Mode**: Support dark mode via CSS variables scoped to a `.dark` parent class. Do not write component-level dark mode conditionals.
- **Typography**: Use semantic typography tokens (e.g., `text-foreground`, `font-heading`, `text-sm`). Do not use raw font sizes or families directly in components.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nexvyn/Nexvyn-ui](https://github.com/Nexvyn/Nexvyn-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
