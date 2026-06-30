---
trigger: always_on
description: This is a Backstage Q&A plugin that provides questions, answers, articles, collections, and links functionality within Backstage. The plugin consists of multiple packages in a monorepo structure:
---

# Copilot Instructions for backstage-plugin-qeta

## Project Overview

This is a Backstage Q&A plugin that provides questions, answers, articles, collections, and links functionality within Backstage. The plugin consists of multiple packages in a monorepo structure:

### Plugin Packages

- **`@drodil/backstage-plugin-qeta`** - Frontend plugin with React components
- **`@drodil/backstage-plugin-qeta-backend`** - Backend plugin with REST API and database layer
- **`@drodil/backstage-plugin-qeta-common`** - Shared types and utilities
- **`@drodil/backstage-plugin-qeta-react`** - Shared React components and hooks
- **`@drodil/backstage-plugin-qeta-node`** - Node utilities
- **`@drodil/backstage-plugin-qeta-backend-module-openai`** - AI integration module
- **`@drodil/backstage-plugin-catalog-backend-module-qeta`** - Catalog integration module
- **`@drodil/backstage-plugin-search-backend-module-qeta`** - Search integration module
- **`@drodil/backstage-plugin-scaffolder-backend-module-qeta`** - Scaffolder integration module

## Technology Stack

- **Framework**: Backstage.io
- **Language**: TypeScript
- **Frontend**: React 19, Material-UI v4
- **Backend**: Node.js (20 or 22)
- **Database**: PostgreSQL (primary), SQLite (development)
- **Build Tool**: Backstage CLI, Yarn workspaces
- **Testing**: Jest (via Backstage CLI)
- **Linting**: ESLint (via Backstage CLI)

## Code Style and Conventions

### General Guidelines

1. **Follow Backstage patterns**: Use Backstage's core components, utilities, and conventions
2. **TypeScript**: Use strict TypeScript with proper typing, avoid `any`
3. **Conventional Commits**: All commit messages must follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
4. **Backward Compatibility**: Maintain backward compatibility unless it's a major version bump

### Frontend Code

- Use **React hooks** and functional components
- Import Backstage components from `@backstage/core-components`
- Use **Material-UI v4** (not v5) - imports from `@material-ui/core`
- Follow the plugin's routing pattern with route refs from `@drodil/backstage-plugin-qeta-react`
- Use the `QetaApi` from context for API calls
- Components should be in `plugins/qeta/src/components/`

### Backend Code

- Use the **new backend system** (not legacy backend)
- Database queries should use the database abstraction from `@backstage/backend-common`
- Services are in `plugins/qeta-backend/src/service/`
- Database layer in `plugins/qeta-backend/src/database/`
- Support both PostgreSQL and SQLite databases
- Use proper database migrations in `plugins/qeta-backend/migrations/`

### Common Patterns

- **Shared types** go in `qeta-common` package
- **React hooks** go in `qeta-react` package
- **API client** should be in `qeta-common`
- Use `@backstage/catalog-model` for entity references
- Integrate with Backstage's permission framework when `permissions` is enabled

## Testing Requirements

- Write tests for new functionality
- Run `yarn test` to execute tests
- Run `yarn test:all` for coverage reports
- Tests should be co-located with source files (`.test.ts`, `.test.tsx`)
- Ensure `yarn lint` and `yarn tsc` pass before committing

## Database Considerations

- Use migrations for schema changes in `plugins/qeta-backend/migrations/`
- Database queries should work with both PostgreSQL and SQLite
- Use parameterized queries to prevent SQL injection

## Development Workflow

### Local Development

```bash
yarn install              # Install dependencies
yarn docker:up            # Start PostgreSQL in Docker
yarn dev                  # Start both frontend and backend
yarn start                # Frontend only
yarn start-backend        # Backend only
```

### Before Committing

```bash
yarn lint                          # Lint all files
yarn tsc                           # Type check
yarn test                          # Run tests
yarn prettier:check                # Check formatting
yarn commitlint --from=origin/main # Commit message check
yarn dedupe                        # Deduplicate dependencies
```

## Integration Points

The plugin integrates with several Backstage systems:

- **Search**: Indexes questions, articles, and answers for search
- **Catalog**: Attaches posts to catalog entities
- **Permissions**: Optional permission framework integration
- **Notifications**: Sends notifications for various events
- **Signals**: Real-time updates using Backstage signals
- **Home**: Provides home page cards
- **Scaffolder**: Creates scaffolder actions

## Configuration

Configuration lives in `app-config.yaml` under the `qeta` key. Key config areas:

- `allowAnonymous`: Allow anonymous posting
- `permissions`: Enable Backstage permission framework
- `entities`: Configure entity attachment rules
- `tags`: Configure tagging behavior
- `storage`: Configure image storage (database, filesystem, S3, Azure)
- `notifications`: Configure notification behavior

See `docs/config.md` for full configuration reference.

## Important Notes

1. **MUI Version**: This plugin uses MUI4, not MUI5. Always import from `@material-ui/core`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drodil/backstage-plugin-qeta](https://github.com/drodil/backstage-plugin-qeta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
