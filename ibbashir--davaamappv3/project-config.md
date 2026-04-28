---
trigger: always_on
description: Davaam App V3 is a **role-based dashboard management application** for dispensing machines (Butterfly, Cooking Oil, Cleaning Products). Built with React 19 + TypeScript + Vite. It supports six user roles: **superadmin, admin, ops, fulfill, finance, company**.
---

# CLAUDE.md — Davaam App V3

## Project Overview

Davaam App V3 is a **role-based dashboard management application** for dispensing machines (Butterfly, Cooking Oil, Cleaning Products). Built with React 19 + TypeScript + Vite. It supports six user roles: **superadmin, admin, ops, fulfill, finance, company**.

## Quick Start

```bash
npm run dev        # Start Vite dev server
npm run build      # Production build (tsc -b && vite build)
npm run lint       # ESLint check
npm run preview    # Preview production build
```

## Path Alias

`@/*` maps to `./src/*` (configured in tsconfig.json and vite.config.ts).

```ts
import { Button } from "@/components/ui/button";
```

## Project Structure

```
src/
├── Apis/              # Axios-based API layer (Api.ts, Authorization.ts)
├── Corporate/         # Company-specific feature modules
├── Types/             # Shared TypeScript type definitions
├── assets/            # Images, videos, GIFs, SVGs
├── components/        # UI components organized by role + shared ui/
│   ├── ui/            # shadcn/ui primitives (button, input, dialog, etc.)
│   ├── admin/         # Admin-specific components
│   ├── superAdmin/    # SuperAdmin components
│   ├── finance/       # Finance components
│   ├── ops/           # Ops components
│   ├── fulfillment/   # Fulfillment components
│   ├── corporate/     # Corporate components
│   ├── app-sidebar.tsx
│   ├── nav-main.tsx
│   └── login-form.tsx
├── constants/         # Route constants, sidebar config, icons, BASE_URL
├── contexts/          # AuthContext (useReducer pattern)
├── hooks/             # Custom hooks (useIsMobile)
├── layouts/           # Layout wrapper (sidebar + Outlet)
├── lib/               # Utility functions (cn = clsx + tailwind-merge)
├── routing/           # React Router setup, PrivateRouting guard
├── screens/           # Page-level components organized by role
│   ├── superAdmin/
│   ├── admin/
│   ├── ops/
│   ├── finance/
│   ├── fulfill/
│   └── corporate/
├── utils/             # Session management utilities
├── App.tsx            # Main app with providers
├── main.tsx           # Entry point
└── index.css          # Tailwind CSS imports + design tokens
```

## Tech Stack

| Category | Technology |
|---|---|
| Framework | React 19 + TypeScript 5.8 |
| Build | Vite 6.3 |
| Styling | Tailwind CSS 4.1 + shadcn/ui (New York style) |
| Routing | React Router DOM v7 |
| State | Context API + useReducer (AuthContext) |
| HTTP | Axios with interceptors |
| Forms | react-hook-form + zod validation |
| Tables | @tanstack/react-table v8 |
| Charts | Recharts, Nivo, Chart.js |
| Maps | Leaflet / react-leaflet, Google Maps |
| Icons | Tabler Icons, Lucide, React Icons, MUI Icons |
| Notifications | sonner, react-toastify |

## API Layer

All API calls go through `src/Apis/Api.ts` which provides generic typed methods:

```ts
getRequest<T>(endpoint: string): Promise<T>
postRequest<T>(endpoint: string, data: object): Promise<T>
putRequest<T>(endpoint: string, data: object): Promise<T>
deleteRequest<T>(endpoint: string): Promise<T>
```

**Authorization** (`src/Apis/Authorization.ts`):
- Axios instance with `withCredentials: true` for cookie-based auth
- Request interceptor adds access token from cookies
- Response interceptor handles 401/403 with automatic token refresh via `/auth/refresh`
- Base URL: `https://davaam-backend-nodejs-4199d6d4d449.herokuapp.com/api/dashboard`

## Authentication Flow

1. Login via `POST /auth/login` → returns user data + accessToken + refreshToken
2. Tokens stored in cookies (path: `/`), access token also kept in memory
3. `AuthContext` uses useReducer with actions: `LOGIN`, `LOGOUT`, `LOADED`
4. On mount, checks cookies for existing session via `POST /auth/user`
5. Auto-refresh on 401/403 responses via interceptor
6. `useAuth()` hook exposes `{ user, token, loading, login, logout }`

## Routing & Role-Based Access

Routes are protected via `<PrivateRouting allowedRoles={["role"]} />`:

```tsx
<Route element={<PrivateRouting allowedRoles={["superadmin"]} />}>
  <Route element={<Layout />}>
    <Route path="/superadmin/dashboard" element={<SuperAdminDashboard />} />
  </Route>
</Route>
```

- Route path constants defined in `src/constants/Constant.ts` (UPPERCASE_SNAKE_CASE)
- URL pattern: `/<role>/<feature>` (e.g., `/superadmin/dashboard`, `/admin/machines`)
- Public routes: `/login`, `/forgetPassword`, `/reset-password`, `/company-info`, `/privacypolicy`
- Sidebar navigation configured per-role as functions returning nav item arrays

## Styling Conventions

- **Tailwind CSS** with OKLch color tokens defined in `src/index.css`
- **shadcn/ui** components in `src/components/ui/` (do not edit directly — use `npx shadcn@latest add`)
- **`cn()` utility** from `src/lib/utils.ts` for merging Tailwind classes:
  ```ts
  import { cn } from "@/lib/utils";
  cn("base-class", conditional && "conditional-class")
  ```

## Component Patterns

- **File naming**: kebab-case (`login-form.tsx`, `app-sidebar.tsx`)
- **Exports**: Named exports for reusable components, default exports for page/screen components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ibbashir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
