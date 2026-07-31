---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UnpodWeb is a Next.js 16 social communication platform built with Nx monorepo architecture. The application features AI agents, video conferencing (LiveKit), real-time messaging, knowledge bases, and SIP bridge integrations.

The project includes a **Tauri desktop application** (`apps/unpod-tauri`) that wraps the Next.js app in a lightweight native desktop experience.

## Commands

### Development
```bash
# Start dev server for the main app
npx nx dev web

# Start desktop app (Tauri) - recommended for desktop development
npm run desktop:dev

# Build production bundle
npx nx build web

# Build desktop app for production
npm run desktop:build

# Start production server (requires build first)
npx nx start web

# Show all available targets for a project
npx nx show project web

# Visualize project graph
npx nx graph
```

### Desktop App (Tauri)
```bash
# Development with hot reload
npm run desktop:dev

# Production build (creates installers)
npm run desktop:build

# Debug build (faster, includes symbols)
npm run desktop:build:debug

# Regenerate app icons
npm run desktop:icon
```

### Linting
```bash
# Lint the main app
npx nx lint web

# Lint with auto-fix
npx nx lint web -- --fix
```

### Testing
Note: This project currently has no test configuration (`unitTestRunner: "none"` in nx.json generators). Tests should be added if implementing new features that require validation.

## Architecture

### Monorepo Structure

This is an Nx monorepo with workspace organization:
- **apps/web** - Main Next.js application (web & desktop frontend)
- **apps/unpod-tauri** - Tauri desktop application (Rust backend + native wrapper)
- **libs/** - Shared libraries organized by domain

### Library Organization

The codebase uses 14 shared libraries with clear separation of concerns:

1. **libs/nextjs/providers** - React Context providers for state management
   - `AuthContextProvider` - User authentication, active hub, subscriptions
   - `AppContextProvider` - Locale and theme settings
   - `AppInfoViewProvider` - Global notifications and loading states
   - `AppOrgContextProvider` - Organization/workspace context
   - `AppModuleContextProvider` - Feature-specific state
   - `APIHooks` - Custom hooks for API calls (useGetDataApi, postDataApi, etc.)

2. **libs/nextjs/services** - HTTP clients and external service integrations
   - `httpClient` - Main Axios instance with JWT auth (points to apiUrl)
   - `httpLocalClient` - Local API client for token management
   - Firebase configuration

3. **libs/nextjs/components** - Shared React components
   - Common UI components exported via wildcard

4. **libs/nextjs/modules** - Complex feature modules
   - AppAgentModule, AppAgentStudio, AppIdentityAgentModule
   - AppKnowledgeBase, AppPost, AppThread
   - AppSIPBridge, AppSpace, AppSubscription, AppWallet

5. **libs/nextjs/custom-hooks** - Custom React hooks for reusable logic

6. **libs/nextjs/helpers** - Utility functions organized by domain
   - ApiHelper, DateHelper, FormHelper, GlobalHelper
   - PermissionHelper, StringHelper, TableHelper, UrlHelper

7. **libs/nextjs/constants** - Application-wide constants

8. **libs/icons** - SVG icon components

9. **libs/localization** - Internationalization (react-intl)

10. **libs/nextjs/livekit** - Video conferencing SDK integration

11. **libs/nextjs/skeleton** - Loading skeleton components

12. **libs/nextjs/mix** - Shared theme configuration and utilities

13. **libs/nextjs/external-libs** - Third-party library integrations

14. **libs/nextjs/react-data-grid** - Data grid components and utilities

### Routing Structure

Next.js App Router with group-based layouts in `apps/web/src/app/`:

- **(auth-layout)/** - Authentication pages (signin, signup, password reset, create-org, join-org)
- **(front-layout)/** - Public landing pages (privacy policy, terms, contact)
- **(main-layout)/** - Protected application pages
  - **(sidebar)/** - Pages with sidebar (ai-studio, agent-studio, bridges, spaces)
  - **(full-layout)/** - Full-width pages (dashboard, ai-agents, threads, profile, billing, org)
  - **[orgSlug]/** - Dynamic organization routes

Route groups (parentheses) isolate layouts without affecting URL paths.

### State Management

The application uses **React Context API with useReducer** (not Redux). All providers follow this pattern:

```javascript
// State split into separate contexts
const [state, dispatch] = useReducer(reducer, initialState);
<StateContext.Provider value={state}>
  <ActionsContext.Provider value={actions}>
```

Access state via hooks:
```javascript
const { user, activeOrg } = useAuthUser();
const { updateLocale } = useAppActionsContext();
```

### API Integration Pattern

All API calls use custom hooks from `libs/nextjs/providers/src/lib/APIHooks.js`:

**Fetching data:**
```javascript
// Paginated GET request
const [{ loading, apiData, page, hasMoreRecord }, { setPage, setData }] =
  usePaginatedDataApi('/agents', [], { orgSlug: 'my-hub' });

// Simple GET request
const [{ loading, apiData }, { setData, setQueryParams }] =
  useGetDataApi('/spaces/123');
```

**Mutations:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geneffic/unpod](https://github.com/geneffic/unpod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
