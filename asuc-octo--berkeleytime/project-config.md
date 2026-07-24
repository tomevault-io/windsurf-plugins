---
trigger: always_on
description: Berkeleytime is a course discovery and planning tool for UC Berkeley students. It combines academic information from UC Berkeley's Office of the Registrar and Engineering Integration Services (EIS) to provide students with course data, schedules, ratings, and planning tools.
---

# Berkeleytime Codebase Rules

## Project Overview

Berkeleytime is a course discovery and planning tool for UC Berkeley students. It combines academic information from UC Berkeley's Office of the Registrar and Engineering Integration Services (EIS) to provide students with course data, schedules, ratings, and planning tools.

## Architecture

This is a **monorepo** using Turbo for build orchestration. The codebase is organized into:

- **Apps**: All stored in the `app/` directory (backend, frontend, datapuller are the main ones)
- **Packages**: Shared code in `packages/` (theme, common, gql-typedefs, sis-api, shared, etc.)
- **Infrastructure**: Kubernetes/Helm charts in `infra/`

## Tech Stack

### Backend (`apps/backend`)
- **Runtime**: Node.js (>=22.12.0) with TypeScript
- **Framework**: Express.js
- **API**: Apollo GraphQL Server
- **Database**: MongoDB (Mongoose ORM)
- **Cache**: Redis
- **Auth**: Passport.js

### Frontend (`apps/frontend`)
- **Framework**: React with TypeScript
- **Build Tool**: Vite
- **GraphQL Client**: Apollo Client
- **Styling**: SCSS Modules
- **UI Components**: Custom components from `@repo/theme` (Radix UI based)
- **Routing**: React Router

### Other Services
- **Datapuller**: Service for pulling data from UC Berkeley APIs
- **Docs**: mdBook documentation site
- **Staff/AG Frontends**: Admin interfaces

## Code Organization Patterns

### Backend Modules (`apps/backend/src/modules/`)

Backend follows a **module-based architecture**. Each module represents a domain (User, Course, Rating, etc.) and follows this structure:

```
modules/[domain]/
  ├── typedefs/           # GraphQL schema definitions
  ├── generated-types/    # Generated TypeScript types (DO NOT EDIT)
  ├── controller.ts       # Database querying functions (Mongoose)
  ├── formatter.ts        # Optional: DB models → GraphQL types
  ├── resolver.ts         # GraphQL resolvers
  └── index.ts            # Module entrypoint
```

**Pipeline**: Request → Resolver → Controller → Database → Formatter → Response

- **DO NOT** edit files in `generated-types/` directories - they are auto-generated
- Use GraphQL Code Generation: run `npm run generate` in backend after schema changes
- Controllers use Mongoose to query MongoDB
- Formatters translate DB models to GraphQL types (only if types differ)

### Frontend Structure (`apps/frontend/src/`)

```
src/
  ├── app/                # Views, pages, and scoped components
  ├── components/         # Reusable React components
  ├── contexts/           # React contexts
  ├── hooks/              # React hooks
  ├── lib/                # Utility functions and logic
  │   └── api/            # GraphQL queries, mutations, and types
  └── utils/              # Utility functions
```

- Use **SCSS Modules** for component styling: `[Component].module.scss`
- Component folders: `index.tsx` + `[Component].module.scss`
- Child components should be extracted when logic becomes complex or reusable
   - Reusable Berkeleytime-specific components (such as a ClassCard) should be put in `components/`
   - Reusable generic components (like a Select) should be put in the parent `packages/theme/src/components` folder

## Conventions

### TypeScript
- Use strict TypeScript
- Generate types from GraphQL schemas - don't write them manually for API query related types
- Use generated types from `generated-types/` directories

### GraphQL
- Overall schemas defined in `packages/gql-typedefs`
- **Backend**: Resolvers are defined in in `resolver.ts`
- **Frontend**: Define queries/mutations in `lib/api/`, use Apollo Client hooks
- Always run `npm run generate` after GraphQL schema changes
- Use GraphQL Code Generation for type safety

### Styling
- Use SCSS Modules (`.module.scss`) for component styles
- Use Bootstrap utility classes (`m-*`, `p-*`) for simple spacing when appropriate
- Import styles: `import styles from "./Component.module.scss"`
- Use `classNames` utility (or similar) when combining classes instead of manual string manipulation (e.g., `classNames(styles.base, { [styles.active]: isActive })` instead of `styles.base + (isActive ? ' ' + styles.active : '')`)
- Use/extend `@repo/theme` components when available instead of building custom ones

### File Naming
- Components: PascalCase (`RatingButton.tsx`)
- Utilities/hooks: camelCase (`ratingErrorMessages.ts`, `useReadClass.ts`)
- SCSS modules: `[Component].module.scss`

## Shared Packages

- **`@repo/theme`**: UI component library (Radix UI primitives + Berkeleytime-specific)
- **`@repo/common`**: Shared utilities and types
- **`@repo/gql-typedefs`**: Shared GraphQL type definitions
- **`@repo/sis-api`**: Auto-generated TypeScript clients for UC Berkeley APIs
- **`@repo/shared`**: Shared business logic

Use these packages rather than duplicating code across apps.

## Development Workflow

1. **Type Generation**: Always run `npm run generate` after GraphQL changes
2. **Type-Check**: Uses `tsc` (`npm run type-check`)
3. **Linting**: Use ESLint (configured in `packages/eslint-config`, `npm run lint`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asuc-octo/berkeleytime](https://github.com/asuc-octo/berkeleytime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
