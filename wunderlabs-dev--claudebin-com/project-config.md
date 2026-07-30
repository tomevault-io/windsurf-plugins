---
trigger: always_on
description: ├── app/             # Next.js App Router pages
---

# Claudebin Web - TypeScript UI Engineering Standards

## Project Structure

```
src/
├── app/             # Next.js App Router pages
├── components/      # Reusable UI components
│   └── ui/          # shadcn/ui components
├── sections/        # Page-level section components
├── utils/           # Utilities, constants, helpers
├── copy/            # i18n translations (en-EN.json)
├── i18n/            # i18n config
└── static/          # CSS, fonts
```

## Critical Rules

### NEVER Use Tailwind Arbitrary Values

```tsx
// Wrong
<div className="w-[347px] text-[#ff0000] mt-[23px]" />

// Correct
<div className="w-full max-w-md text-destructive mt-6" />
```

**If you need a custom value:** Add it to `globals.css` @theme first.

### ALWAYS Use `const`

```tsx
// Correct
const items = data.items;

// Wrong
let items = data.items;
```

Use `let` only when reassignment is absolutely necessary.

### Import Ordering

```tsx
// 1. CSS imports
import "@/static/css/globals.css";

// 2. Vendors (React, third-party)
import { useRef } from "react";

// 3. Custom hooks
import { useUiState } from "@/hooks/useUiState";

// 4. UI components (shadcn/ui)
import { Button } from "@/components/ui/Button";

// 5. Custom components
import { HomeIntro } from "@/components/home-intro";

// 6. Sections
import { SessionSection } from "@/sections/SessionSection";
```

Each group separated by blank line. Always use `@/*` path alias.

## Component Conventions

### Structure

Components are single files using kebab-case naming (no directories):

```
components/
├── home-intro.tsx
├── home-tutorials.tsx
├── ui/           # shadcn/ui components
└── icon/         # Icon components
```

### Arrow Functions Only

```tsx
// Correct
export const DashboardAppBar = () => { ... }

// Wrong
export function DashboardAppBar() { ... }
```

### Props

- Use `type` (never `interface`)
- Spread native HTML attributes: `& HTMLAttributes<HTMLElement>`
- Set defaults in parameters: `variant = "default"`

```tsx
type ButtonProps = {
  variant?: ButtonVariant;
} & HTMLButtonAttributes<HTMLButtonElement>;

export const Button = ({ variant = "default", ...props }: ButtonProps) => { ... }
```

### Type Patterns

Use `as const` for variant unions:

```typescript
export const ButtonVariants = ["default", "outline", "ghost"] as const;
export type ButtonVariant = (typeof ButtonVariants)[number];
export type ButtonVariantMapping = Record<ButtonVariant, string>;
```

Use discriminated unions when variant affects required props:

```typescript
type ChipDefaultProps = ChipBaseProps & {
  variant?: "default";
  color: ChipColor;
};
type ChipOutlinedProps = ChipBaseProps & { variant: "outlined"; color?: never };
export type ChipProps = ChipDefaultProps | ChipOutlinedProps;
```

### Exports

Components export directly from their file:

```typescript
// home-intro.tsx
export const HomeIntro = () => { ... };
```

### Event Handlers

Named handlers only, never inline callbacks:

```tsx
// Wrong
<Button onClick={() => console.log("clicked")}>Click</Button>;

// Correct
const handleClick = () => {
  console.log("clicked");
};

<Button onClick={handleClick}>Click</Button>;
```

Naming: `handle[Action]` (e.g., `handleSignUp`, `handleSubmit`)

### Naming Conventions

- **Files**: kebab-case for components (`home-intro.tsx`), camelCase for utils (`helpers.ts`)
- **Types**: `ComponentNameProps`, `ComponentNameVariant`, `ComponentNameVariantMapping`
- **Mapping objects**: `xxxClassNames` (e.g., `buttonVariantClassNames`)
- **Const arrays**: Plural form (`ButtonVariants`)

## Styling

### Variant Mapping Objects

```typescript
const buttonVariantClassNames: ButtonVariantMapping = {
  default: "bg-primary text-primary-foreground",
  outline: "border border-input bg-background",
  ghost: "hover:bg-accent hover:text-accent-foreground"
};

// Use cn() for class merging
className={cn("base-classes", buttonVariantClassNames[variant])}
```

### Tailwind Rules

- Reference `globals.css` @theme for all values
- Use semantic color names (`primary`, `secondary`, `muted`)
- Use spacing scale (`gap-4`, `p-6`, `mt-8`)
- Check `globals.css` @theme before styling anything

### Tailwind Class Ordering

Order classes by visual hierarchy (outside-in, structure to details):

1. **Layout** - display, position, flex/grid setup (`flex`, `grid`, `absolute`, `relative`)
2. **Flex/Grid modifiers** - alignment, justify, direction (`items-center`, `justify-between`, `flex-col`)
3. **Sizing** - width, height, max/min (`w-full`, `max-w-6xl`, `h-screen`)
4. **Spacing** - margin, padding, gap (`mx-auto`, `p-2`, `gap-8`)
5. **Background** - colors, gradients, images (`bg-gray-100`, `bg-radial`)
6. **Border** - border, rounded (`border`, `border-gray-500/20`, `rounded-3xl`)
7. **Typography** - font, text, leading (`text-xl`, `font-bold`, `leading-normal`)
8. **Effects** - shadow, opacity, blur (`shadow-lg`, `opacity-50`)
9. **Transitions** - transition, duration, ease (`transition`, `duration-200`)
10. **Selectors** - child/descendant selectors last (`[&_video]:w-full`)

```tsx
// Correct ordering
<div className="flex items-center w-full max-w-6xl mx-auto p-2 bg-radial from-gray-100/50 to-gray-200/50 border border-gray-500/20 rounded-3xl [&_video]:w-full [&_video]:rounded-2xl" />

// Categories visible:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wunderlabs-dev/claudebin.com](https://github.com/wunderlabs-dev/claudebin.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
