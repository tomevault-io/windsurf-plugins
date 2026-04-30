---
trigger: always_on
description: This guide helps AI agents work effectively in the Endurance Coach codebase - a TypeScript/Svelte application for creating personalized training plans.
---

# Endurance Coach - Agent Guide

This guide helps AI agents work effectively in the Endurance Coach codebase - a TypeScript/Svelte application for creating personalized training plans.

## Project Overview

Endurance Coach is a dual-component system:

- **CLI Tool**: Generates training plans from compact YAML templates using a domain-specific language
- **Web Viewer**: Single-page Svelte application embedded in rendered HTML for viewing and managing plans

The system uses a contract-first, template-based architecture where AI agents compose plans from predefined workout templates rather than generating raw JSON.

## Essential Commands

### Development

```bash
# Run CLI directly (equivalent to npx endurance-coach)
npm start -- [command] [options]

# Watch mode for CLI development
npm run dev

# Development server for web viewer (Svelte)
npm run dev:viewer

# Run all tests (watch mode)
npm test

# Run tests once
npm run test:run

# Test all workout templates can be converted
npm run test:allTemplates

# Expand compact plan to JSON
npm run test:expandPlanToJson
```

### Building

```bash
# Build everything (TypeScript + viewer + skill package)
npm run build

# Build only TypeScript
npm run build:ts

# Build only web viewer
npm run build:viewer

# Package skill for distribution
npm run build:skill
```

### Code Quality

```bash
# Type check without emitting
npm run typecheck

# Format all code with Prettier
npm run format

# Check formatting (fails if not formatted)
npm run format:check
```

### Testing Workflow

After making changes:

1. Run `npm run typecheck` to ensure types are correct
2. Run `npm test` to run tests in watch mode
3. Run `npm run format:check` before committing (pre-commit hook enforces this)

**When Modifying Templates:**

After adding or modifying workout templates, also run:

```bash
npm run test:allTemplates
```

This test renders a plan that includes every built-in template to ensure all templates can be converted properly. If a template has conversion issues (e.g., invalid duration format), the test will fail and indicate which template is problematic.

## Project Structure

```
src/
├── cli.ts              # Command-line interface (main entry point)
├── index.ts            # Public API exports
├── db/                 # SQLite database schema and client
│   ├── schema.sql      # Database schema and views
│   ├── client.ts       # SQLite connection and query functions
│   └── migrate.ts     # Database migration logic
├── lib/                # Shared utilities
│   ├── config.ts      # Configuration management
│   └── logging.ts     # CLI logging utilities
├── schema/             # Type schemas and validation (Zod)
│   ├── training-plan.schema.ts    # v1.0 full JSON schema
│   ├── compact-plan.schema.ts     # v2.0 YAML template schema
│   ├── training-plan.ts           # TypeScript types
│   └── compact-plan.ts           # Compact plan utilities
├── templates/          # Template loading and parsing
│   ├── loader.ts      # Load templates from filesystem
│   ├── yaml-parser.ts # YAML parsing wrapper
│   ├── interpolate.ts # Template variable interpolation
│   ├── converter.ts   # Convert template structures to workout structures (for device export)
│   └── index.ts      # Public API exports
├── expander/           # Plan expansion logic
│   ├── expander.ts    # Core expansion (compact → full)
│   ├── zones.ts       # Heart rate, power, pace zone calculations
│   └── types.ts       # Expanded plan types
├── strava/             # Strava API integration
│   ├── api.ts         # Activity and athlete endpoints
│   ├── oauth.ts       # OAuth token management
│   └── types.ts       # Strava API types
└── viewer/             # Web-based plan viewer (Svelte 5)
    ├── App.svelte      # Main application
    ├── main.ts         # Entry point
    ├── components/     # Svelte components
    ├── lib/           # Export utilities (ICS, FIT, ZWO, ERG)
    └── stores/        # Svelte 5 runes state management

templates/              # Workout template definitions (YAML)
├── run/               # Running workouts
├── bike/              # Cycling workouts
├── swim/              # Swimming workouts
├── strength/          # Strength training
└── brick/             # Brick workouts (bike+run)

tests/                 # Test files (Vitest)
├── cli/               # CLI tests
└── viewer/            # Viewer tests
```

## Code Conventions

### TypeScript

- Use ES2022 target with `NodeNext` module resolution
- Strict mode enabled
- All exports must include type definitions
- Use `.js` extensions in imports (required by NodeNext)

### Svelte

- **Svelte 5 Runues**: Use `$state`, `$effect`, `$derived` for reactivity
  ```typescript
  let settings = $state(loadSettings());
  $effect(() => {
    document.documentElement.setAttribute("data-theme", settings.theme);
  });
  ```
- TypeScript with `<script lang="ts">` blocks
- Components use PascalCase (e.g., `WorkoutModal.svelte`)

### File Naming

- TypeScript files: kebab-case (e.g., `yaml-parser.ts`)
- Svelte components: PascalCase (e.g., `WorkoutModal.svelte`)
- Templates: lowercase with hyphens (e.g., `tempo.yaml`)

### Prettier Configuration

- Use double quotes (`"`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiv19/endurance-coach-skill](https://github.com/shiv19/endurance-coach-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
