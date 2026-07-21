---
trigger: always_on
description: All code is in the `src/` directory. The legacy `source/` directory has been fully migrated.
---

# ATMN CLI - Code Conventions

## Directory Structure

All code is in the `src/` directory. The legacy `source/` directory has been fully migrated.

```
atmn/
├── src/
│   ├── cli.tsx                # Main CLI entry point
│   ├── constants.ts           # BACKEND_URL, FRONTEND_URL, DEFAULT_CONFIG
│   │
│   ├── compose/               # DSL builders for plans/features (public API)
│   │   ├── index.ts           # Package exports (Plan, Feature, plan(), feature(), etc.)
│   │   ├── models/            # Type definitions (Plan, Feature, PlanFeature)
│   │   └── builders/          # Builder functions (plan(), feature(), planFeature())
│   │
│   ├── commands/              # CLI commands (each in its own folder)
│   │   ├── auth/              # Auth/login command
│   │   ├── pull/              # Pull command
│   │   ├── push/              # Push command
│   │   └── nuke/              # Nuke command
│   │
│   ├── lib/                   # Shared libraries
│   │   ├── api/               # API client and endpoints
│   │   │   ├── client.ts      # HTTP client
│   │   │   ├── endpoints/     # API endpoint functions
│   │   │   └── types/         # API response types
│   │   ├── hooks/             # Custom React hooks
│   │   ├── transforms/        # Data transformation (apiToSdk, sdkToApi, sdkToCode)
│   │   ├── env/               # Environment context (--local, --prod flags)
│   │   ├── constants/         # Shared constants, template data
│   │   └── utils.ts           # Shared utilities
│   │
│   └── views/                 # UI templates
│       ├── html/              # HTML templates (OAuth callbacks)
│       └── react/             # React/Ink components
│           ├── components/    # Shared UI components
│           ├── init/          # Init flow
│           ├── pull/          # Pull UI
│           ├── push/          # Push UI
│           └── nuke/          # Nuke UI
│
├── test/                      # Tests
└── dist/                      # Build output
```

## Package Exports

The `atmn` package exports the compose DSL for use in `autumn.config.ts`:

```ts
// Users import from 'atmn'
import { plan, feature, planFeature, type Plan, type Feature } from 'atmn';
```

These exports come from `src/compose/index.ts`.

## Architecture Conventions

### Commands (`src/commands/<name>/`)

Each command should have its own folder with:
- Main command file(s) with the command logic
- Supporting files for complex operations (e.g., `oauth.ts` for auth)
- Types and constants specific to that command

### Views (`src/views/`)

UI templates organized by type:
- `html/` - HTML templates (for browser callbacks, etc.)
- `react/` - React/Ink components organized by feature

### Shared Libraries (`src/lib/`)

- `api/` - API client, endpoint functions, and response types
- `hooks/` - Custom React hooks (useOrganization, usePush, usePull, etc.)
- `transforms/` - Data transformation functions between API/SDK/Code formats
- `env/` - CLI context (--local, --prod flag handling)
- `constants/` - Shared constants and template data
- `utils.ts` - Shared utility functions

## Import Conventions

- Use `.js` extensions in imports (required for ESM)
- Import compose types from `../../compose/models/index.js`
- Import API functions from `../../lib/api/endpoints/index.js`
- Import utilities from `../../lib/utils.js`

## OAuth Flow

The auth command uses OAuth 2.1 PKCE:
1. Local HTTP server starts on port `31448` (or next available up to `31452`)
2. Browser opens to backend authorize URL with PKCE challenge
3. User authenticates and selects organization
4. Callback received with authorization code
5. Code exchanged for access token
6. Access token used to create API keys via `/cli/api-keys` endpoint
7. Keys saved to `.env` file

Key constants:
- `CLI_CLIENT_ID`: `qncNuaPFAEBwzCypjFopNCGPHQDqkchp`
- `OAUTH_PORTS`: `31448-31452` (5 ports, tries each in sequence if previous is in use)

## React Component Conventions (`src/views/react/`)

When React files grow large, use folder-based decomposition:

1. **Create a folder with the component's name** - Replace `Component.tsx` with `Component/` folder
2. **Move parent into folder** - Place main component in `Component/Component.tsx`
3. **Extract children as siblings** - Create separate files for sub-components (e.g., `Component/SubComponent.tsx`)
4. **Apply recursively** - If children grow large, repeat the pattern for them
5. **Extract shared components** - Move frequently reused components to `react/components/`

**Example:**
```
react/
├── InitFlow.tsx              # Original file (300+ lines)
│
# After decomposition:
├── init/
│   ├── InitFlow.tsx          # Main orchestrator
│   └── steps/
│       ├── AuthStep.tsx      # Step 1
│       ├── StripeStep.tsx    # Step 2
│       └── ConfigStep.tsx    # Step 3
│
└── components/               # Shared across multiple views
    ├── StepHeader.tsx
    ├── StatusLine.tsx
    └── SelectMenu.tsx
```

**Useful Ink Components:**
- `ink-select-input` - Interactive select menus with arrow key navigation
- `ink-confirm-input` - Yes/No confirmation prompts
- `ink-spinner` - Loading spinners
- `ink-text-input` - Text input fields

## Build

```bash
pnpm build    # Build with Bun
pnpm dev      # Watch mode
pnpm test     # Run tests
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [useautumn/autumn](https://github.com/useautumn/autumn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
