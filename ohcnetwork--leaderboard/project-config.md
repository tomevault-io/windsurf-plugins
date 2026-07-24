---
trigger: always_on
description: This is a flexible, plugin-based leaderboard system for tracking and visualizing contributor activities across multiple data sources. The system follows a build-time data aggregation pattern with static site generation.
---

# Leaderboard Project - AI Assistant Rules

## Project Overview

This is a flexible, plugin-based leaderboard system for tracking and visualizing contributor activities across multiple data sources. The system follows a build-time data aggregation pattern with static site generation.

### Architecture
- **Pattern**: Plugin-based with static site generation
- **Data Flow**: Data Sources → Plugin Runner → LibSQL Database → Next.js Build → Static Site
- **Deployment**: Static export to CDN (Netlify, Vercel, GitHub Pages, etc.)

### Technology Stack
- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js v20+
- **Package Manager**: pnpm v10+ (monorepo with workspaces)
- **Frontend**: Next.js 14+ (static export only)
- **Database**: LibSQL (SQLite-compatible)
- **Documentation**: Fumadocs (MDX)
- **UI Components**: shadcn/ui + Tailwind CSS
- **Testing**: Vitest
- **Module System**: ESNext (ESM with `.js` extensions required)

## Monorepo Structure

```
leaderboard/
├── apps/
│   └── leaderboard-web/          # Next.js static site
├── packages/
│   ├── api/                      # @ohcnetwork/leaderboard-api
│   ├── plugin-runner/            # @leaderboard/plugin-runner
│   ├── plugin-dummy/             # @leaderboard/plugin-dummy
│   ├── create-plugin/            # create-leaderboard-plugin
│   └── create-data-repo/         # create-leaderboard-data-repo
├── docs/                         # Documentation (MDX)
├── scripts/                      # Development scripts
└── data/                         # Development data repository
```

### Key Packages

1. **@ohcnetwork/leaderboard-api**
   - Database utilities and abstractions
   - Plugin type definitions and interfaces
   - Query builders (contributorQueries, activityQueries, etc.)
   - Shared types and schemas

2. **@leaderboard/plugin-runner**
   - CLI tool for orchestrating data collection
   - Plugin loading and execution
   - Import/export functionality
   - Aggregation and badge evaluation

3. **create-leaderboard-plugin**
   - CLI for scaffolding new plugins
   - Generates template with tests and docs

4. **create-leaderboard-data-repo**
   - CLI for initializing data repositories
   - Interactive setup for organization config
   - Generates proper directory structure

5. **leaderboard-web**
   - Next.js application (static export)
   - Server-side generation at build time
   - Reads from LibSQL database

## Coding Conventions

### TypeScript
- **Strict Mode**: Always enabled
- **Module System**: ESNext with ESM
- **Type Safety**: Avoid `any` unless absolutely necessary; use `unknown` instead
- **Interfaces vs Types**: Use `interface` for object shapes, `type` for unions/intersections

### Naming Conventions
- **Files/Directories**: kebab-case (`activity-loader.ts`, `badge-rules/`)
- **Functions/Variables**: camelCase (`getUserActivities`, `totalPoints`)
- **Types/Interfaces/Classes**: PascalCase (`ActivityDefinition`, `PluginContext`)
- **Constants**: SCREAMING_SNAKE_CASE for true constants (`MAX_RETRIES`)
- **Private Members**: Prefix with `_` (`_internalCache`)

### File Organization
- **Source Code**: `src/` directory
- **Tests**: `src/__tests__/` directory
- **Test Files**: `{module-name}.test.ts`
- **Types**: Co-locate with implementation or in `types.ts`
- **Exports**: Use named exports (avoid default exports except for plugins and Next.js pages)

### Database Patterns
- **Query Builders**: Always use provided query builders from `@ohcnetwork/leaderboard-api`
  ```typescript
  // ✅ Good
  import { contributorQueries } from "@ohcnetwork/leaderboard-api";
  const user = await contributorQueries.getByUsername(db, "alice");
  
  // ❌ Bad (use only when query builders don't cover the use case)
  await db.execute("SELECT * FROM contributor WHERE username = ?", ["alice"]);
  ```
- **Transactions**: Use `db.batch()` for multiple related operations
- **Parameterization**: Always use parameterized queries (never string concatenation)

### Error Handling
- **Async Functions**: Always use try-catch or .catch()
- **Logging**: Use structured logger provided in context
  ```typescript
  try {
    await riskyOperation();
  } catch (error) {
    logger.error("Operation failed", error, { context: "additional-info" });
    throw error; // Re-throw if caller should handle
  }
  ```
- **User-Facing Errors**: Provide clear, actionable error messages

## Key Terminology

### Core Concepts
- **Plugin**: JavaScript/TypeScript module that fetches data from external sources (GitHub, Slack, etc.)
- **Contributor**: User with a profile stored as Markdown file with YAML frontmatter
- **Activity**: Single tracked event or contribution (PR, issue, comment, etc.)
- **Activity Definition**: Type of activity defined by plugins (e.g., "pr_merged", "issue_opened")
- **Data Repository**: Separate git repository containing config.yaml, contributors/, activities/
- **Aggregate**: Computed metric (total_activities, activity_count, longest_streak, etc.)
- **Badge**: Achievement or reward earned based on rule evaluation
- **Rule**: Badge eligibility criteria (streak, count, total_points)

### Data Storage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohcnetwork/leaderboard](https://github.com/ohcnetwork/leaderboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
