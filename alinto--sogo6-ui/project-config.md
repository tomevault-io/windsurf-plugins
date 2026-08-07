---
trigger: always_on
description: This file provides context and guidelines for AI coding agents working on this project.
---

# AGENTS.md - AI Coding Agent Configuration

This file provides context and guidelines for AI coding agents working on this project.

## Project Overview

**SOGo6-UI** is a modern web-based groupware frontend for the SOGo server, providing email, calendar, and address book functionality. It's built with Next.js 16 and React 19.

## Tech Stack

| Category             | Technology                                             |
| -------------------- | ------------------------------------------------------ |
| Framework            | Next.js 16 (App Router, Turbopack)                     |
| Language             | TypeScript (strict mode)                               |
| UI Library           | React 19                                               |
| Styling              | Tailwind CSS, tailwind-merge, class-variance-authority |
| Components           | shadcn/ui (Radix UI primitives)                        |
| State Management     | Redux Toolkit (RTK)                                    |
| Forms                | React Hook Form + Zod validation                       |
| Internationalization | next-intl                                              |
| Testing              | Jest + React Testing Library                           |
| Linting              | ESLint + Prettier                                      |
| Drag & Drop          | @dnd-kit                                               |
| Calendar             | react-big-calendar                                     |
| Rich Text Editor     | CKEditor 5                                             |

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── [locale]/           # Internationalized routes
│   │   ├── (admin)/        # Admin panel routes
│   │   ├── (auth)/         # Authentication routes
│   │   ├── (loggedin)/     # Protected user routes
│   │   └── (others)/       # Other routes
│   ├── env/                # Environment API route
│   └── fakeApi/            # Mock API endpoints for development
├── components/             # Shared React components
│   ├── ui/                 # shadcn/ui base components
│   ├── calendar/           # Calendar-specific components
│   ├── sidebar/            # Sidebar components
│   └── dnd/                # Drag and drop components
├── features/               # Feature-based modules (domain logic)
│   ├── address_books/      # Address book feature
│   ├── admin-panel/        # Admin panel feature
│   ├── auth/               # Authentication feature
│   ├── calendars/          # Calendar feature
│   ├── mails/              # Email feature
│   ├── notifications/      # Notifications feature
│   ├── themes/             # Theme management
│   └── user-settings/      # User preferences
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities and configurations
│   ├── auth/               # Auth utilities
│   ├── i18n/               # i18n configuration
│   └── redux/              # Redux store setup
├── messages/               # Translation files (JSON)
│   └── en/                 # English translations
└── types.ts                # Shared TypeScript types
```

## Coding Conventions

### File Naming

- Use **kebab-case** for file names: `list-item-mobile.tsx`
- Use `.tsx` for React components, `.ts` for utilities
- Test files: `__tests__/component-name.test.tsx`

### Component Patterns

- Use **functional components** with hooks
- Export components as named exports
- Place component-specific types in the same file or a sibling `types.ts`

### Imports

- Use the `@/` path alias for imports from `src/`:
  ```typescript
  import { Button } from '@/components/ui/button'
  import { useAppSelector } from '@/lib/redux/hooks'
  ```

### Exports

- use memo() for components that do not always need to re-render
- memo is declared at the export default line
  ```typescript
  export default memo(MyComponent)
  ```
- use export default only for React components

### Styling

- Use Tailwind CSS utility classes
- Use `cn()` utility from `@/lib/utils` for conditional classes:
  ```typescript
  import { cn } from "@/lib/utils";
  className={cn("base-class", condition && "conditional-class")}
  ```
- Use `cva` (class-variance-authority) for component variants

### State Management

- Use Redux Toolkit for global state
- Use React Hook Form for form state
- Use local state (`useState`) for UI-only state

### Internationalization

- **NEVER hardcode user-facing strings**
- Use `useTranslations()` hook from `next-intl`:
  ```typescript
  const t = useTranslations("mails");
  return <span>{t("inbox")}</span>;
  ```
- Translation keys must be static strings (enforced by ESLint)
- Add new translations to `src/messages/en/*.json`

### Forms & Validation

- Use React Hook Form with Zod schemas:
  ```typescript
  const schema = z.object({ email: z.string().email() })
  const form = useForm({ resolver: zodResolver(schema) })
  ```

## Testing Guidelines

### Running Tests

```bash
npm test              # Run all tests with coverage
npm run test:fast     # Quick run without coverage
npm run test:watch    # Watch mode
npm run test:changed  # Only changed files
```

### Test Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alinto/SOGo6-UI](https://github.com/Alinto/SOGo6-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
