---
trigger: always_on
description: This document provides context and guidelines for AI agents working on the Squirrel-UI codebase.
---

# Squirrel-UI AI Agent Context

This document provides context and guidelines for AI agents working on the Squirrel-UI codebase.

## 🎯 Project Overview

Squirrel-UI is a React-based **Personal Expense Tracker**. Users manage transactions (income/expense), categorize them, and view financial summaries via charts. Currency is **INR** (`en-IN` locale).

## 🛠️ Tech Stack

| Layer | Library |
|-------|---------|
| UI framework | React 19 (functional components + hooks) |
| CSS | **Tabler CSS 1.2** (Bootstrap-based — use Bootstrap utility classes) |
| Icons | `@tabler/icons-react` |
| Dialogs | `@radix-ui/react-dialog`, `@radix-ui/react-alert-dialog` |
| Charts | Recharts |
| Routing | React Router DOM v7 |
| Server state | **TanStack Query v5** (`@tanstack/react-query`) |
| Forms | **React Hook Form v7** + **Zod v4** (`@hookform/resolvers`) |
| Mocking | **MSW v2** (tests + optional dev mode) |
| Build | Vite 7 |
| Testing | Vitest + Testing Library |

> **No Tailwind.** Use Tabler/Bootstrap utility classes (`d-flex`, `gap-2`, `text-secondary`, `min-vh-100`, etc.).

## 📁 Source Structure

```
src/
  api/           auth.js · categories.js · transactions.js
  components/    AppLayout.jsx · ErrorBoundary.jsx · Notification.jsx
  context/       NotificationContext.jsx · ThemeContext.jsx
  hooks/         useCategories.js · useTransactions.js
  infra/
    auth/        guards.jsx · storage.js
    http/        client.js · errors.js
    router/      index.jsx
    config.js
  mocks/         handlers.js · server.js · browser.js
  pages/         LoginPage.jsx · DashboardPage.jsx · CategoriesPage.jsx
  test/          setup.js · *.test.jsx
  ui/            barrel — import everything from here (see below)
```

## 🧩 UI Barrel (`src/ui/`)

**Always import from `@/ui` (absolute) or `../ui` (relative). Never from individual files.**

```js
import {
  Button, Card, CardHeader, CardTitle, CardBody,
  Badge, Spinner, FormField, Input, Select,
  Alert, Modal, ConfirmDialog,
  IconPlus, IconEdit, IconTrash, /* …other Tabler icons */
  CHART_COLORS, TRANSACTION_COLORS,
} from '@/ui';
```

Swapping the underlying library only requires changes inside `src/ui/` — zero app-code changes.

## 📐 Absolute Imports

`@/` maps to `src/`. Use in all new files.

```js
import { useCategories } from '@/hooks/useCategories';
import { storage }       from '@/infra/auth/storage';
import { Button }        from '@/ui';
```

## 🗺️ Routes

| Path | Page | Guard |
|------|------|-------|
| `/login` | `LoginPage` | `PublicRoute` |
| `/dashboard` | `DashboardPage` | `PrivateRoute` |
| `/categories` | `CategoriesPage` | `PrivateRoute` |
| `/` | redirect → `/dashboard` or `/login` | — |

## 🌐 HTTP Layer

`src/infra/http/client.js` — thin `fetch` wrapper.
- `apiRequest(path, opts)` — authenticated API calls (attaches Bearer token).
- `authRequest(path, opts)` — auth-service calls (login, refresh).
- **Refresh token**: on 401, silently calls `config.refreshPath`, retries original request. Concurrent 401s queue — only one refresh fires.
- Falls back to `/login` redirect if refresh fails or no refresh token stored.
- To swap `fetch` → `axios`: edit `client.js` only.

## 🔑 Auth Storage

`src/infra/auth/storage.js` — single point for localStorage keys.
- `storage.getToken()` / `setToken()` — access token.
- `storage.getRefreshToken()` / `setRefreshToken()` — refresh token.
- `storage.getUser()` / `setUser()` — parsed user object (`{ id, email, role, permissions[] }`).
- `storage.clear()` — clears all three.
- Swap localStorage → cookie: edit this file only.

## 🛡️ Auth Guards

```jsx
import { PrivateRoute, PublicRoute, RequireRole, RequirePermission } from '@/infra/auth/guards';

// Route-level
<PrivateRoute><Dashboard /></PrivateRoute>
<PublicRoute><LoginPage /></PublicRoute>

// Component-level role check
<RequireRole role="admin" fallback={<p>No access</p>}>
  <AdminPanel />
</RequireRole>

// Component-level permission check
<RequirePermission permission="transactions:write">
  <CreateButton />
</RequirePermission>
```

`user.role` (string) and `user.permissions` (string[]) come from `storage.getUser()`.

## 🔁 TanStack Query — Data Fetching Pattern

**All server state goes through TanStack Query. No manual `useState` + `useEffect` for fetching.**

```js
// Query (read)
const { data, isLoading, error } = useQuery({
  queryKey: ['resource', filters],
  queryFn: () => fetchResource(filters),
  select: (res) => res.data,        // transform here, not in component
});

// Mutation (write)
const mutation = useMutation({
  mutationFn: (payload) => createResource(payload),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['resource'] });
    showNotification('Created!', 'success');
  },
  onError: (err) => showNotification(err.message, 'error'),
});
```

Query keys: `[RESOURCE_KEY]` (all) or `[RESOURCE_KEY, filters]` (filtered).
Export the key constant from the hook file: `export const CATEGORIES_KEY = 'categories'`.

Global config in `App.jsx`: `staleTime: 30_000` (30s fresh window), `retry: 1`.

## 📋 Forms — React Hook Form + Zod

**All forms use RHF + Zod. No per-field `useState`. No manual validation.**

```js
import { useForm } from 'react-hook-form';
import { z } from 'zod';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zooarium/squirrel-ui](https://github.com/zooarium/squirrel-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
