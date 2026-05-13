---
trigger: always_on
description: This is an **Untitled UI React** component library project built with:
---

## Project Overview

This is an **Untitled UI React** component library project built with:

- **React 19** with TypeScript
- **Tailwind CSS v4.2** for styling
- **React Aria Components** as the foundation for accessibility and behavior

## Key Architecture Principles

### Component Foundation

- All components are built on **React Aria Components** for consistent accessibility and behavior
- Components follow the compound component pattern with sub-components (e.g., `Select.Item`, `Select.ComboBox`)
- TypeScript is used throughout for type safety

### Import Naming Convention

**CRITICAL**: All imports from `react-aria-components` must be prefixed with `Aria*` for clarity and consistency:

```typescript
// ✅ Correct
import { Button as AriaButton, TextField as AriaTextField } from "react-aria-components";
// ❌ Incorrect
import { Button, TextField } from "react-aria-components";
```

This convention:

- Prevents naming conflicts with custom components
- Makes it clear when using base React Aria components
- Maintains consistency across the entire codebase

### File Naming Convention

**IMPORTANT**: All files must be named in **kebab-case** for consistency:

```
✅ Correct:
- date-picker.tsx
- user-profile.tsx
- api-client.ts
- auth-context.tsx

❌ Incorrect:
- DatePicker.tsx
- userProfile.tsx
- apiClient.ts
- AuthContext.tsx
```

This applies to all file types including:

- Component files (.tsx, .jsx)
- TypeScript/JavaScript files (.ts, .js)
- Style files (.css, .scss)
- Test files (.test.ts, .spec.tsx)
- Configuration files (when creating new ones)

## Development Commands

```bash
# Development
npm run dev               # Start Vite development server (http://localhost:5173)
npm run build            # Build for production (TypeScript compilation + Vite build)
```

## Project Structure

### Application Architecture

```
src/
├── components/
│   ├── base/              # Core UI components (Button, Input, Select, etc.)
│   ├── application/       # Complex application components
│   ├── foundations/       # Design tokens and foundational elements
│   ├── marketing/         # Marketing-specific components
│   └── shared-assets/     # Reusable assets and illustrations
├── hooks/                 # Custom React hooks
├── pages/                 # Route components
├── providers/             # React context providers
├── styles/               # Global styles and theme
├── types/                # TypeScript type definitions
└── utils/                # Utility functions
```

### Component Patterns

#### 1. Base Components

Located in `components/base/`, these are the building blocks:

- `Button` - All button variants with loading states
- `Input` - Text inputs with validation and icons
- `Select` - Dropdown selections with complex options
- `Checkbox`, `Radio`, `Toggle` - Form controls
- `Avatar`, `Badge`, `Tooltip` - Display components

#### 2. Application Components

Located in `components/application/`, these are complex UI patterns:

- `DatePicker` - Calendar-based date selection
- `Modal` - Overlay dialogs
- `Pagination` - Data navigation
- `Table` - Data display with sorting
- `Tabs` - Content organization

#### 3. Styling Architecture

- Uses a `sortCx` utility for organized style objects
- Follows size variants: `sm`, `md`, `lg`, `xl`
- Color variants: `primary`, `secondary`, `tertiary`, `destructive`, etc.
- Responsive and state-aware styling with Tailwind

#### 4. Component Props Pattern

```typescript
interface CommonProps {
    size?: "sm" | "md" | "lg";
    isDisabled?: boolean;
    isLoading?: boolean;
    // ... other common props
}

interface ButtonProps extends CommonProps, HTMLButtonElement {
    color?: "primary" | "secondary" | "tertiary";
    iconLeading?: FC | ReactNode;
    iconTrailing?: FC | ReactNode;
}
```

## Styling Guidelines

### Tailwind CSS v4.2

- Uses the latest Tailwind CSS v4.2 features
- Custom design tokens defined in theme configuration
- Consistent spacing, colors, and typography scales

### Brand Color Customization

To change the main brand color across the entire application:

1. **Update Brand Color Variables**: Edit `src/styles/theme.css` and modify the `--color-brand-*` variables
2. **Maintain Color Scale**: Ensure you provide a complete color scale from 25 to 950 with proper contrast ratios
3. **Example Brand Color Scale**:
    ```css
    --color-brand-25: rgb(252 250 255); /* Lightest tint */
    --color-brand-50: rgb(249 245 255);
    --color-brand-100: rgb(244 235 255);
    --color-brand-200: rgb(233 215 254);
    --color-brand-300: rgb(214 187 251);
    --color-brand-400: rgb(182 146 246);
    --color-brand-500: rgb(158 119 237); /* Base brand color */
    --color-brand-600: rgb(127 86 217); /* Primary interactive color */
    --color-brand-700: rgb(105 65 198);
    --color-brand-800: rgb(83 56 158);
    --color-brand-900: rgb(66 48 125);
    --color-brand-950: rgb(44 28 95); /* Darkest shade */
    ```

The color scale automatically adapts to both light and dark modes through the CSS variable system.

### Style Organization

```typescript
export const styles = sortCx({
    common: {
        root: "base-classes-here",
        icon: "icon-classes-here",
    },
    sizes: {
        sm: { root: "small-size-classes" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [untitleduico/react](https://github.com/untitleduico/react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
