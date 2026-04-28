---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Important**: Always read `[AGENTS.md](./AGENTS.md)` alongside this file. AGENTS.md documents the current state of the repository and provides practical guidance for coding agents. This file (CLAUDE.md) provides Claude Code-specific development instructions and deeper architectural context.

## Development Commands

### Core Development

- `npm run dev` - Start concurrent frontend (Vite) and backend (nodemon) development servers
- `npm run dev-up` - Kill ports 3001/5173/8000 and start development servers (preferred for clean startup)
- `npm run client:dev` - Frontend only (Vite dev server on port 5173)
- `npm run server:dev` - Backend only (Express API on port 3001)
- `npm run build` - TypeScript check + Vite build for production
- `npm run lint` - ESLint validation
- `npm run check` - TypeScript type checking without emitting files
- `npm run preview` - Preview production build locally

### API Documentation

- `npm run docs:api:sync` - Sync API documentation (auto-runs via pre* hooks on `dev`, `client:dev`, and `build`)
- `npm run docs:api:audit` - Audit and validate API documentation coverage

### Security

- `npm run audit:security` - Run npm audit with high severity filter
- `npm run scan:code` - Run semgrep static analysis
- `npm run test:security` - Run security tests with Vitest
- `npm run verify:security` - Run all security checks (audit + scan + test)

### Testing

> **Note**: There is no `npm test` script. Run `npx vitest run tests/security/` for the security test suite, or `npx vitest run` for all tests. The repo includes Vitest, React Testing Library, Supertest, and Playwright.

### Database Management

- `npm run db:fresh` - Reset database and run all migrations (WARNING: destroys all data - use only in development)
- `npm run db:reset` - Interactive database reset with confirmation
- `npm run db:reset:confirm` - Reset database without prompt
- `npm run seed:admin` - Create default admin user
- `node scripts/seed-branding.js` - Update database branding (docs, FAQ, contact, rDNS) to match .env

### Production Deployment

- `npm run start` - Launch production Express server + Vite preview
- `npm run pm2:start` - Build and start with PM2 process manager
- `npm run pm2:reload` - Reload PM2 processes gracefully
- `npm run pm2:stop` - Stop PM2 processes
- `npm run pm2:list` - List PM2 processes

### Utilities

- `npm run kill-ports` - Kill processes on ports 3001, 5173, and 8000
- `npm run pwa:icons` - Generate PWA icons

## Architecture Overview

SkyPanelV2 is a full-stack cloud service reseller billing panel with multi-tenant organization support. It provides three distinct product surfaces:

1. **Public marketing pages** — Home, pricing, FAQ, about, contact, status, legal, docs, regions
2. **Authenticated customer portal** — Dashboard, VPS management, billing, support, SSH console, organizations, egress credits, API docs
3. **Admin dashboard** — User management, billing ops, platform settings, provider config, impersonation, documentation, announcements, networking

### Frontend (`src/`)

- **React 18** SPA with TypeScript and Vite
- **shadcn/ui** component library with Tailwind CSS (Radix UI primitives), configured via `components.json`
- **TanStack Query** for server state management with optimistic updates (30s staleTime, 1 retry, refetchOnWindowFocus)
- **Zustand** for client state management (selected stores for UI state)
- **React Router v7** for routing with protected route guards:
  - `ProtectedRoute`: Auth required → AppLayout with sidebar
  - `StandaloneProtectedRoute`: Auth required → AppLayout without sidebar (SSH console)
  - `AdminRoute`: Auth + `user.role === 'admin'` + not impersonating → AppLayout (admin dashboard)
  - `PublicRoute`: Redirects to `/dashboard` if already logged in
- **React Hook Form + Zod** for form validation with schema-based validation
- **Framer Motion** for animations with accessibility considerations
- **Recharts** for data visualization with responsive containers
- **xterm.js** for browser-based SSH terminal emulation with fit/addon packages
- **cmdk** for command palette (Ctrl/Cmd + K) with persistent state
- **TipTap** for rich text editing with code highlighting (lowlight)
- **qrcode** for 2FA QR code generation using otplib

### Backend (`api/`)

- **Express 4** REST API with TypeScript ESM modules
- **PostgreSQL** database with UUID primary keys, JSONB columns, triggers, and `LISTEN/NOTIFY`
- **JWT authentication** with role-based access (admin/user) and HttpOnly cookie storage
- **Rate limiting** with tiered configuration (anonymous/authenticated/admin) and per-user overrides stored in database
- **CSRF protection** on `/api` routes
- **Comprehensive middleware** stack:
  - `security.ts`: Helmet, CSP with nonce, CORS, referrer policy, XSS protection
  - `auth.ts`: JWT verification, token blacklist, organization validation, impersonation context
  - `permissions.ts`: Role-based and organization membership checking (single/any/all permission variants)
  - `rateLimiting.ts`: Tiered limits with X-RateLimit headers, metrics, Redis support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gvps-cloud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
