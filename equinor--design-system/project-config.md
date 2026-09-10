---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the @equinor/eds-mobile-components package.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the @equinor/eds-mobile-components package.

## Package Overview

This package is the core React Native component library implementing the Equinor Design System. It provides UI components with full support for light/dark theming and comfortable/spacious density modes.

## Local Development

From this package directory:

```bash
# Build the component library
pnpm build

# Watch mode for development (auto-rebuilds on changes)
pnpm dev

# Lint
pnpm lint

# Type check
pnpm types

# Clean build artifacts
pnpm clean
```

From the monorepo root, the equivalents are `pnpm build:mobile`, `pnpm watch:mobile`, and `pnpm check-types:mobile` (hand-sequenced to build this package first). There's no root-level lint equivalent — root's `eslint.config.mjs` ignores this package outright (ESLint's flat config doesn't cascade per-directory, so linting it from the root would otherwise apply root's Prettier-style rules — single quotes, no semicolons — to code written in this package's own style). `pnpm lint:all` exits 0 on these files (with an easy-to-miss "File ignored" warning) rather than actually checking them. Use `pnpm --filter @equinor/eds-mobile-components run lint` (or run it from this directory) to get this package's own config.

## Architecture

### Token and Theming System

The library uses a token-based theming system sourced from `@equinor/eds-tokens`:

**Token Sources:**

- `theme.colors.*` — color tokens (`ColorToken`)
- `theme.spacing.*` — spacing tokens (`SpacingToken`)
- `theme.typography.*` — typography tokens (`TypographyToken`)
- `theme.geometry.*` / `theme.timing.*` — static constants (no `@equinor/eds-tokens` equivalent yet)

**Theme Context:**

- `EDSProvider` wraps the app and provides theme context via React Context
- Accepts `colorScheme` ("light" | "dark") and `density` ("comfortable" | "spacious")
- Creates a `MasterToken` that resolves theme values based on current scheme/density
- Internally wraps app in `GestureHandlerRootView`, `PortalProvider`, `ScrimProvider`, and `DialogServiceProvider`

**Creating Theme-Aware Styles:**

```tsx
// Define styles using EDSStyleSheet.create
const themeStyles = EDSStyleSheet.create((token) => ({
    container: {
        backgroundColor: token.colors.bg.neutral.surface,
        padding: token.spacing.spacing.inset.lg.horizontal,
    },
}));

// Resolve styles in component using useStyles hook
const MyComponent = () => {
    const styles = useStyles(themeStyles);
    return <View style={styles.container} />;
};
```

**Conditional Styling with Props:**

```tsx
const themeStyles = EDSStyleSheet.create(
    (token, props: { highlighted?: boolean }) => ({
        container: {
            backgroundColor: props.highlighted
                ? token.colors.bg.accent.fillEmphasis.default
                : token.colors.bg.neutral.surface,
        },
    })
);

const MyComponent = ({ highlighted }: { highlighted?: boolean }) => {
    const styles = useStyles(themeStyles, { highlighted });
    return <View style={styles.container} />;
};
```

**Direct Token Access:**

- Use `useToken()` hook to access token values directly in component logic
- Prefer `EDSStyleSheet` + `useStyles` for styling
- Only use `useToken()` when you need token values for non-style purposes

### Directory Structure

```
src/
├── components/        # All UI components (Button, Paper, TextField, Dialog, etc.)
├── hooks/            # Shared hooks (useEDS, useStyles, useToken, useBreakpoint, etc.)
├── styling/          # Theming system (EDSStyleSheet, token types, color/spacing)
│   └── tokens/       # Token type definitions
├── utils/            # Utility functions and types
└── assets/           # Fonts and static assets
```

Each component typically exports:

- Main component (e.g., `Button`)
- Type definitions (e.g., `ButtonProps`)
- Sub-components if applicable (e.g., `Dialog.Alert`, `Dialog.Confirm`)

### Build System

**Build Configuration:**

- Uses `tsup` for bundling (ESM format, tree-shaking enabled)
- `tsc` generates TypeScript declarations separately
- Entry: All `.ts/.tsx` files in `src/` (excluding tests and type definitions)
- Output: `dist/` directory with bundled JS and declaration files
- Font assets (`.otf`) are copied to `dist/assets/fonts/`

**Build Process:**

1. `tsup` bundles source code with splitting and tree-shaking
2. `tsc --emitDeclarationOnly` generates `.d.ts` files
3. In watch mode (`pnpm dev`), both steps run automatically on file changes

### Key Dependencies

**Peer Dependencies** (must be installed by consumers):

- `react`, `react-dom`, `react-native` - Core React Native
- `react-native-gesture-handler` - Touch gestures
- `react-native-reanimated` - Animations
- `react-native-svg` - SVG support
- `expo-font` - Font loading

**Internal Dependencies:**

- `@equinor/eds-tokens` - Design token source
- `@floating-ui/react-native` - Popover positioning
- `react-error-boundary` - Error boundary utilities

### Font Loading

Consumers must call `useEDS()` hook before rendering components to load required fonts:

```tsx
export default function App() {
    const [hasLoadedEds, edsLoadError] = useEDS();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [equinor/design-system](https://github.com/equinor/design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
