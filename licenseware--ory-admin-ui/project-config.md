---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working on this codebase.
---

# CLAUDE.md - Ory Admin UI

This document provides comprehensive guidance for AI assistants working on this codebase.

## Project Overview

Ory Admin UI is a production-grade single-page application for managing identities and access control in Ory Kratos identity servers. It provides an intuitive interface for CRUD operations on identities, sessions, courier messages, and identity schemas.

**Key Features:**

- Identity management (create, read, update, delete)
- Session management and revocation
- Courier message viewing
- Identity schema browsing
- Dark/light theme support
- Responsive design

## Tech Stack

| Category         | Technology                                      |
| ---------------- | ----------------------------------------------- |
| Framework        | Vue 3.5 (Composition API with `<script setup>`) |
| Language         | TypeScript 5.7 (strict mode)                    |
| Build Tool       | Vite 6                                          |
| Package Manager  | Bun (primary) or npm                            |
| State Management | Pinia                                           |
| Data Fetching    | TanStack Vue Query                              |
| HTTP Client      | ky                                              |
| Styling          | Tailwind CSS 3.4                                |
| UI Components    | Radix Vue primitives                            |
| Icons            | Lucide Vue Next                                 |
| Notifications    | vue-sonner                                      |
| Linting          | oxlint, ESLint                                  |
| Formatting       | Prettier                                        |
| Testing          | Vitest                                          |

## Project Structure

```
src/
├── api/                 # API layer - HTTP clients for Ory Kratos
│   ├── client.ts        # Base ky client configuration
│   ├── identities.ts    # Identity CRUD operations
│   ├── sessions.ts      # Session management
│   ├── courier.ts       # Courier message API
│   ├── schemas.ts       # Identity schemas API
│   └── health.ts        # Health check endpoints
├── assets/
│   └── styles/
│       └── main.css     # Global styles and Tailwind imports
├── components/
│   ├── common/          # Reusable utility components
│   │   ├── CopyButton.vue
│   │   ├── EmptyState.vue
│   │   ├── ErrorState.vue
│   │   ├── JsonViewer.vue
│   │   ├── LoadingSpinner.vue
│   │   ├── Pagination.vue
│   │   ├── StatusBadge.vue
│   │   └── TimeAgo.vue
│   ├── layout/          # App shell components
│   │   ├── AppFooter.vue
│   │   ├── AppHeader.vue
│   │   ├── AppShell.vue
│   │   └── AppSidebar.vue
│   └── ui/              # Base UI primitives (shadcn-vue style)
│       ├── Button.vue
│       ├── Card.vue, CardContent.vue, CardHeader.vue, etc.
│       ├── Dialog.vue
│       ├── Input.vue
│       ├── Select.vue
│       ├── Tabs.vue
│       └── ...
├── composables/         # Vue composables for data fetching
│   ├── useCourier.ts
│   ├── useHealth.ts
│   ├── useIdentities.ts
│   ├── useSchemas.ts
│   └── useSessions.ts
├── lib/
│   └── utils.ts         # Utility functions (cn, formatDate, etc.)
├── router/
│   └── index.ts         # Vue Router configuration
├── stores/              # Pinia stores
│   ├── settings.ts      # API endpoint configuration
│   ├── theme.ts         # Theme (dark/light/system)
│   └── ui.ts            # UI state (sidebar, modals)
├── types/
│   └── api.ts           # TypeScript interfaces for API models
├── views/               # Route-level page components
│   ├── DashboardView.vue
│   ├── IdentitiesView.vue
│   ├── IdentityDetailView.vue
│   ├── IdentityCreateView.vue
│   ├── SessionsView.vue
│   ├── SessionDetailView.vue
│   ├── CourierView.vue
│   ├── SchemasView.vue
│   ├── SettingsView.vue
│   └── NotFoundView.vue
├── App.vue              # Root component
├── main.ts              # Application entry point
└── vite-env.d.ts        # Vite type declarations
```

## Development Workflow

### Commands

```bash
# Install dependencies
bun install           # or: npm install

# Development server (default: http://localhost:5173)
bun run dev           # or: npm run dev

# Type checking
bun run typecheck     # or: npm run typecheck

# Build for production
bun run build         # or: npm run build

# Preview production build
bun run preview       # or: npm run preview

# Linting
bun run lint          # oxlint

# Formatting
bun run format        # Prettier write
bun run format:check  # Prettier check

# Testing
bun run test          # Vitest watch mode
bun run test:coverage # With coverage
```

### Justfile Tasks

```bash
just lint    # Run pre-commit hooks
just serve   # Start dev server
just build   # Production build
just mkdocs  # Serve documentation
```

### Environment Variables

Create `.env` file based on `.env.example`:

```bash
VITE_DEFAULT_API_ENDPOINT=http://localhost:4434  # Kratos Admin API URL
```

The API endpoint can also be configured at runtime in the Settings page.

## Architecture Patterns

### Component Patterns

1. **Composition API with `<script setup>`** - All components use the modern Vue 3 syntax:

   ```vue
   <script setup lang="ts">
   import { ref, computed } from "vue"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [licenseware/ory-admin-ui](https://github.com/licenseware/ory-admin-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
