---
trigger: always_on
description: This file provides instructions for Claude when working on the Wonder Blocks design system codebase.
---

# Wonder Blocks Design System

This file provides instructions for Claude when working on the Wonder Blocks design system codebase.

## Technology Stack

- **Language**: TypeScript (Strict mode enforced)
- **Framework**: React (Functional Components and Hooks)
- **Styling**: Aphrodite (`aphrodite`) for CSS-in-JS with Wonder Blocks tokens (`@khanacademy/wonder-blocks-tokens`)
- **State Management**: Local state (`useState`) and React Context API (`useContext`)
- **Data Fetching**: There should be no data fetching in the design system UI components
- **Routing**: Design system components with link functionality should also support React-Router routes
- **Testing**: Jest, React Testing Library (RTL), `@testing-library/user-event`, Storybook
- **Package Manager**: pnpm

## Coding Conventions

### File & Export Naming

- Use kebab-case for files and directories (e.g., `activity-button.tsx`, `utils.ts`)
- Test files: `*.test.ts(x)`
- Storybook files: `*.stories.tsx`
- PascalCase for React components and types (e.g., `Button`, `type ButtonProps`)
- camelCase for functions, variables, hooks (e.g., `getButtonProps`, `useButtonFunctionality`)

### TypeScript

- Use `strict` mode
- Define clear interfaces/types. Use `type` for props and state, `interface` for shared structures where appropriate
- Use utility types (`Partial`, `Omit`, `Pick`, `Readonly`, etc.)
- Use the `satisfies` operator for type-safe object literals
- Prefer type-only imports: `import type {...}` for types
- **Avoid `any`**; use `unknown` or specific types instead

### React

- **Imports**: Always use `import * as React from "react"` (required for JSX transformation)
- Use functional components and hooks (`useState`, `useEffect`, `useContext`, `useCallback`, `useMemo`)
- Define explicit `Props` types with object destructuring. Pass complex objects/callbacks with stable references (`useCallback`, `useMemo`) if they are dependencies of effects or memoized children
- Keep component state minimal. Lift state up when necessary
- Provide stable `key` props for lists (use item IDs if available)
- Build complex UI by composing smaller Wonder Blocks when possible
- Use `React.forwardRef` when components need to expose DOM refs
- Extract reusable logic into custom hooks (e.g., `useFieldValidation`, `useIsMounted` from `@khanacademy/wonder-blocks-core`)
- Event Handlers: Internal handlers prefixed with `handle` (e.g., `handleClick`), callback props prefixed with `on` (e.g., `onClick`)
- **Don't use `React.FC<Props>`**, use `(props: Props) =>` instead
- **Avoid creating new class components**

**Class to Functional Migration**: When converting class components to functional components, if there is a `componentWillUnmount` method, the corresponding `useEffect` should either not have any dependencies or should call the `isMounted` function returned by `useIsMounted` from `@khanacademy/wonder-blocks-core`.

### Styling (Aphrodite)

- Define styles using `StyleSheet.create` from `aphrodite`. Colocate styles with the component
- Use `addStyle` from `@khanacademy/wonder-blocks-core` to create styled HTML elements
- Use semantic color tokens from `@khanacademy/wonder-blocks-tokens` (e.g., `semanticColor.core.background.base`)
- Use the `focusStyles` utility from `@khanacademy/wonder-blocks-styles` for focus indicators
- **Avoid using primitive `color` tokens**; use `semanticColor` tokens instead

### Wonder Blocks Usage

- Use `View` from `@khanacademy/wonder-blocks-core` for layout containers instead of plain divs
- Use React's `useId` hook to generate unique ids (not the deprecated `Id` component)
- Use `Heading` and `BodyText` from `@khanacademy/wonder-blocks-typography` for text
- Import Phosphor icons from `@phosphor-icons/core` (e.g., `import plusIcon from "@phosphor-icons/core/regular/plus.svg"`) and use `PhosphorIcon` from `@khanacademy/wonder-blocks-icon`
- **Avoid deprecated components** like `Strut`

### Imports

- Organize imports: React, third-party libs, internal absolute paths (`@khan/`, `@khanacademy/`), relative paths (`./`, `../`)
- Use absolute paths for cross-package imports

### Linting & Formatting

- Strictly adhere to ESLint and Prettier
- The project enforces import order: React, third-party libs, then internal imports
- JSDoc comments should be used for complex functions, but TypeScript types are preferred over JSDoc type annotations
- **Never remove existing comments** that are used to provide context
- Run `pnpm lint` before submitting changes

## Component Design

### Composition vs Configuration

- **Configuration (Preferred)**: Components accept props that control rendering
  - Example: `Button` has `startIcon` and `endIcon` props rather than children
  - Use when: Precise control over styling, positioning, or behavior of child elements
- **Composition**: Components accept other components as children
  - Example: `SingleSelect` with `OptionItem` components as children
  - Use when: Component contains many similar items or flexibility in structure is needed

### Controlled vs Uncontrolled

- **Controlled**: Parent passes `value` and `onChange` props (e.g., `TextField`, `TextArea`)
- **Uncontrolled**: State lives in the component itself (e.g., `Accordion`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Khan/wonder-blocks](https://github.com/Khan/wonder-blocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
