---
trigger: always_on
description: You are assisting a Senior Frontend Cloud Architect with 12+ years of experience.
---

You are assisting a Senior Frontend Cloud Architect with 12+ years of experience.

EXPERTISE LEVEL:
- Expert-level React 18+ with advanced patterns (Suspense, Concurrent Features, Server Components)
- Advanced TypeScript with complex type systems and generics
- Cloud architecture patterns (AWS/GCP/Azure)
- Micro-frontends and module federation
- Performance optimization and Core Web Vitals
- Modern build tools (Vite, esbuild, Turbopack)
- Testing strategies (Jest, Vitest, Playwright, Cypress)

CODE & ARCHITECTURE STANDARDS:
- Use TypeScript strict mode throughout the codebase
- Follow SOLID principles, clean architecture, and separation of concerns
- Use container/presentational component patterns; separate business logic (hooks/services) from UI rendering
- Organize components, hooks, and services in feature-based directories (see PR_REVIEW_PROMPTS.md for structure)
- Name files and symbols according to conventions: Container (e.g., AppServiceInfoPanel), Presentation (e.g., AppEndpoint), Hooks (e.g., useAppService), Types (e.g., AppServiceProps)
- Design components and hooks for reusability and composability; prefer generic, well-typed props
- Place business logic in service layers and custom hooks; avoid logic in UI components
- Use dependency injection and composition over inheritance
- Optimize for tree-shaking, code splitting, and performance (memoization, minimal re-renders)
- Use React Query for server state, Context API for global state, and local state only where appropriate
- Ensure all dependencies are listed in useEffect /useMemo/useCallback arrays
- Write self-documenting code with JSDoc for all public APIs and complex logic
- Avoid 'any' types; use strict, reusable types and type guards
- Only export types if they are used outside the file; otherwise, keep types local to the file
- All new features must have unit and integration tests; tests should be isolated, meaningful, and follow codebase structure
- Reference and follow codebase examples for patterns and best practices
- New component files must be camelCased (e.g., MyComponent.tsx)
- Do not create many components unless they are designed for reuse or composability
- Avoid creating interfaces; prefer using Types for type definitions
- Use function declarations for all component definitions (e.g., export function MyComponent({ ... }))
- Do not use arrow functions for component definitions
- Do not use React.FC with type annotations for presentational components
- All function components must be defined using function declarations (see eslint rule: react/function-component-definition)
- All new code must follow the ESLint rules set for the project
- Default to creating generic, reusable, and composable components for similar UI/logic (e.g., DocConfigInput for doc config inputs) instead of duplicating code across multiple components. Prioritize DRY, composability, and consistency.
- Use typography components from 'components/typography' (Heading, Text) instead of native HTML elements (<h1>, <h2>, <p>, text in <div>, etc.)
- For all form implementations, see the "Form Usage Patterns" section below.

# Import Ordering
- ALWAYS follow the ESLint import ordering rules. The project uses automatic import sorting.
- Standard import order (enforced by ESLint):
  1. External libraries (react, react-dom, third-party packages)
  2. @couchbasecloud/* packages (e.g., @couchbasecloud/base-ui)
  3. Local component imports (components/*)
  4. Local page/feature imports (pages/*, partials/*)
  5. Type imports (types/*, type imports with 'type' keyword)
  6. Constants imports (constants/*)
  7. Utility imports (utils/*, hooks/*)
  8. Relative imports (./*, ../)
- Example of correct import order:
  ```typescript
  import { useState } from 'react';
  import { TextField } from '@couchbasecloud/base-ui/components/inputs/text-field';
  import { Button } from 'components/button/button';
  import { Text } from 'components/typography/text';
  import type { MyProps } from 'pages/feature/types';
  import { CONSTANTS } from 'constants/app';
  import makeDataAutoId from 'utils/make-data-auto-id';
  import { helperFunction } from './helper';
  ```
- If ESLint shows "Run autofix to sort these imports!", ALWAYS fix the import order immediately
- Never ignore import ordering warnings - they exist to maintain consistency across the codebase
- Use `type` keyword for type-only imports to enable better tree-shaking: `import type { MyType } from './types'`

# Design System Component Usage
- ALWAYS use design system components instead of native HTML elements
- Use `Button` component for all interactive buttons instead of native <button> elements
- Use `Heading` component for all headings instead of native HTML elements (<h1>, <h2>, etc.)
- Use `Text` component for all text content instead of native HTML elements (<p>, <span>, text in <div>, etc.)
- Use design system components for forms (Input, Select, Checkbox, etc.) instead of native form elements
- Follow the established typography scale and weight conventions:

## Button Component
- **Variants**: primary, secondary, tertiary, tertiary-text, danger, success
- **Usage**: `<Button variant="tertiary" onClick={handleClick}>Click Me</Button>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aashishGitHub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
