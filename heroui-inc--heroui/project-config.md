---
trigger: always_on
description: Instructions for AI agents working with the HeroUI v3 repository.
---

# AGENTS.md

Instructions for AI agents working with the HeroUI v3 repository.

## Repository Overview

HeroUI v3 is a modern React UI library built with **Tailwind CSS v4**, organized as a **pnpm monorepo** managed by **Turborepo**. Components are built on top of [React Aria Components](https://react-spectrum.adobe.com/react-aria/) and follow a compound component pattern similar to Radix UI.

### Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Node.js | 22+ | Runtime |
| pnpm | 10.26.2 | Package manager (via corepack) |
| React | 19+ | UI framework |
| Tailwind CSS | 4.x | Styling |
| TypeScript | 5.x | Type safety |
| Turborepo | 2.x | Build orchestration |
| Storybook | Latest | Component development |
| Vitest | 4.x | Testing |
| React Aria Components | Latest | Accessibility primitives |
| tailwind-variants | Latest | Variant-based styling (includes twMerge) |

### Monorepo Structure

```
/
├── apps/
│   └── docs/              # Documentation site (Next.js + Fumadocs)
├── packages/
│   ├── react/             # Main UI library (@heroui/react)
│   │   ├── src/components/  # All components
│   │   ├── src/utils/       # Shared utilities
│   │   └── scripts/         # Build & codegen scripts
│   ├── styles/            # CSS styles & variants (@heroui/styles)
│   │   └── src/components/  # Per-component .css files
│   ├── standard/          # Shared ESLint, Prettier, TS configs
│   ├── storybook/         # Storybook configuration
│   └── vitest/            # Shared Vitest configurations
├── turbo.json
└── pnpm-workspace.yaml
```

## Commands

| Action | Command |
|---|---|
| Install dependencies | `pnpm i --hoist` |
| Build all packages | `pnpm build` |
| Build specific package | `pnpm build --filter=@heroui/react` |
| Dev (Storybook, port 6006) | `pnpm dev` |
| Dev (Docs site, port 3000) | `pnpm dev:docs` |
| Lint | `pnpm lint` |
| Typecheck | `pnpm typecheck` |
| Test all | `pnpm test` |
| Test one component | `pnpm test button` |
| Format | `pnpm run format` |
| Bump version | `pnpm version:bump` |
| Scaffold a new component | `cd packages/react && pnpm add:component ComponentName` |

## Git Commit Convention

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/) and are validated by Husky + commitlint. Pre-commit also runs `lint-staged`.

```
<type>(<scope>): <message>
```

**Allowed types:** `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`

Examples:

```
feat(components): add select component
fix(button): resolve disabled state not applying
docs: update installation guide
```

## Component Architecture

### File Structure

Each component lives in `packages/react/src/components/<component-name>/`:

```
component-name/
├── component-name.tsx          # Component implementation (uses React Aria)
├── component-name.styles.ts    # Tailwind Variants styling
├── component-name.stories.tsx  # Storybook stories
└── index.ts                    # Barrel exports
```

CSS styles live in `packages/styles/src/components/<component-name>/`.

### Creating a New Component

Always use the scaffold script:

```bash
cd packages/react
pnpm add:component ComponentName
```

Then build to update package.json exports:

```bash
pnpm build
```

### Compound Component Pattern

HeroUI uses a compound component pattern. Each component exports its sub-parts so users can compose and style them independently.

```tsx
// Context shares state/styles across parts
const ComponentContext = createContext<{slots?: ReturnType<typeof componentVariants>}>({});

// Root wraps children with context
const ComponentRoot = forwardRef(({children, className, ...props}, ref) => {
  const slots = useMemo(() => componentVariants({...}), [...]);
  return (
    <ComponentContext value={{slots}}>
      <ReactAriaPrimitive ref={ref} className={composeTwRenderProps(className, slots.base())}>
        {children}
      </ReactAriaPrimitive>
    </ComponentContext>
  );
});

// Child parts consume context
const ComponentItem = forwardRef(({className, ...props}, ref) => {
  const {slots} = useContext(ComponentContext);
  return (
    <ReactAriaPrimitive ref={ref} className={composeTwRenderProps(className, slots?.item())}>
      {props.children}
    </ReactAriaPrimitive>
  );
});
```

Compound components are exported via `Object.assign` as the default export:

```tsx
const CompoundComponent = Object.assign(ComponentRoot, {
  Item: ComponentItem,
  Trigger: ComponentTrigger,
});
export default CompoundComponent;
```

### Export Strategy

```tsx
// Named exports for compound components
export * as ComponentName from "./component-name";

// Direct exports for simple components
export {Component, type ComponentProps} from "./component";

// Always export variants
export {componentVariants, type ComponentVariants} from "./component.styles";
```

### Styling Rules

1. **Styles go in `.styles.ts` files**, never in `.tsx` files. Use `tv()` from `tailwind-variants`.
2. **Import from `tailwind-variants`**, never from `@heroui/standard`.
3. **Never use `twMerge` manually** — `tailwind-variants` already includes it.
4. **Add `"use client"` directive** at the top of every component `.tsx` file.
5. **Display names** follow: `HeroUI.ComponentName` or `HeroUI.Component.SubPart`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heroui-inc/heroui](https://github.com/heroui-inc/heroui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
