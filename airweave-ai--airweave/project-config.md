---
trigger: always_on
description: - **React 18** with TypeScript for type-safe component development
---

# Airweave Frontend Architecture & Guidelines

## Tech Stack & Core Technologies
- **React 18** with TypeScript for type-safe component development
- **Vite** for fast development builds and HMR
- **TailwindCSS** with **ShadCN UI** components for consistent design
- **Radix UI** primitives with **Lucide** icons for accessible components
- **React Router** for client-side routing with file-based organization
- **Zustand** for state management with persistence
- **React Query** for server state and data fetching
- **Auth0** for authentication with custom context wrapper
- **SSE (Server-Sent Events)** for real-time sync progress

## Project Structure
```
frontend/src/
├── components/         # Reusable UI components
│   ├── ui/            # ShadCN UI primitives
│   ├── shared/        # Shared business components
│   └── [feature]/     # Feature-specific components
├── pages/             # Route-level components
├── lib/               # Core utilities and providers
│   ├── api.ts         # API client with auth integration
│   ├── stores/        # Zustand state stores
│   └── auth-context.tsx # Auth provider
├── hooks/             # Custom React hooks
├── services/          # Business logic services
├── config/            # Configuration files
├── constants/         # App constants
└── styles/            # Global CSS styles
    └── toast.css      # Custom toast styling
```

## API Layer (`lib/api.ts`)

The API client is the central hub for all backend communication with sophisticated features:

### Core Features
- **Token Management**: Automatic token injection via provider pattern
- **Request Queuing**: Queues requests while auth initializes
- **Organization Context**: Auto-injects `X-Organization-ID` header
- **Session Tracking**: Auto-injects `X-Airweave-Session-ID` header with PostHog session ID for session replay linking
- **Auto-Retry**: Refreshes token on 401/403 and retries
- **Organization Auto-Switching**: Detects resource org mismatches and switches context
- **Type-Safe Responses**: Returns typed Response objects

### Usage Pattern
```typescript
// Always use relative paths (no /api/v1 prefix)
const response = await apiClient.get('/collections');
const response = await apiClient.post('/source-connections', data);
const response = await apiClient.delete('/api-keys', { id: keyId });
```

### Token Provider Pattern
The API client uses a pluggable token provider set up in `main.tsx`:
```typescript
setTokenProvider({
  getToken: async () => await auth.getToken(),
  clearToken: () => auth.clearToken(),
  isReady: () => auth.isReady()
});
```

## State Management Architecture

### 1. **Organization Store** (`stores/organizations.ts`)
- Manages user organizations with persistence
- Handles organization switching with state cleanup
- Auto-selects best organization (current → primary → first)
- Clears org-specific data on switch (collections, API keys)

### 2. **Collections Store** (`stores/collections.ts`)
- Caches collections (with API-provided `source_connection_summaries` for list views)
- Implements event-driven updates via custom event bus
- Smart caching with force refresh option
- Handles collection CRUD events automatically
- Source connection details for list pages come from `GET /collections` response (no per-card fetching)

### 3. **Sync State Store** (`stores/syncStateStore.ts`)
- Real-time sync progress via SSE
- Manages multiple concurrent subscriptions
- Session storage for progress persistence
- Automatic cleanup and health checks

### 4. **API Keys Store** (`stores/apiKeys.ts`)
- Organization-scoped API key management
- Auto-clears on organization switch

## Authentication Flow

### Auth0 Integration
1. **Provider Hierarchy**:
   ```
   PostHogProvider → ThemeProvider → BrowserRouter → Auth0Provider → AuthProvider → ApiAuthConnector → App
   ```
   - **PostHogProvider**: Initializes PostHog analytics and session tracking (outermost)
   - **ThemeProvider**: Manages dark/light theme persistence
   - **BrowserRouter**: React Router client-side routing
   - **Auth0Provider**: Handles user authentication
   - **AuthProvider**: Manages token lifecycle and auth state
   - **ApiAuthConnector**: Connects auth to API client

2. **Auth Context** (`lib/auth-context.tsx`):
   - Manages Auth0 token lifecycle
   - Provides `getToken()` for API calls
   - Handles dev mode (auth disabled)
   - Token initialization tracking

3. **Token Caching Strategy** (`lib/auth0-provider.tsx`):
   - `getCacheLocation()` selects `memory` for custom Auth0 domains and `localstorage` for standard `*.auth0.com` tenants
   - Custom domains share the app's eTLD+1, so the Auth0 session cookie is first-party — tokens stay in memory only (no XSS surface via localStorage)
   - Standard tenants use a third-party cookie that Safari ITP / Firefox strict / Chrome incognito block, so tokens must be persisted in localStorage
   - `useRefreshTokens={true}` — refresh token rotation is always enabled
   - `useRefreshTokensFallback={true}` — enables iframe-based silent auth as a fallback when the refresh token is unavailable (e.g. after a page refresh with memory cache)

4. **Auth Guard** (`components/AuthGuard.tsx`):
   - Protects routes requiring authentication
   - Initializes organizations on first load

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
