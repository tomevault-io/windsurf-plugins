---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**B2B SaaS Starter** - A production-ready Next.js 16 B2B SaaS application with Stytch B2B authentication, Polar.sh billing, and comprehensive RBAC system. Built for scalability, security, and developer experience.

## Development Commands

- **Development**: `pnpm dev` (uses Turbopack)
- **Build**: `pnpm build`
- **Production**: `pnpm start`
- **Lint**: `pnpm lint`
- **Package Manager**: `pnpm` only

## Tech Stack & Architecture

- **Framework**: Next.js 16.0.10 + App Router
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS
- **Components**: shadcn/ui for consistent, accessible UI components
- **State Management**: TanStack Query (React Query) for server state, Zustand for client state
- **Authentication**: Stytch B2B with magic links, session management, and RBAC
- **Billing**: Polar.sh for subscriptions, usage metering, and webhooks
- **Data Fetching**: Server Actions + TanStack Query + Repository pattern
- **Philosophy**: Production-ready, secure, maintainable, performant

## Key Principles

- **Security First**: Authentication guards, permission checks, subscription validation on every sensitive operation
- **Modern Architecture**: Server Actions for mutations, React Query for caching, Next.js 16 App Router for routing
- **Type Safety**: Strict TypeScript throughout the application with comprehensive type definitions
- **Performance**: Optimized bundle size, fast load times, efficient caching strategies
- **Maintainability**: Clear separation of concerns, consistent patterns, comprehensive documentation

## Project Structure

```
app/
├── layout.tsx                    # Root layout with providers
├── page.tsx                      # Landing page
├── auth/                         # Authentication pages
├── authenticate/                 # Magic link callback
├── signup/                       # Organization signup
├── dashboard/                    # Protected dashboard routes
│   ├── page.tsx                 # Dashboard home (redirects to settings)
│   ├── settings/                # Settings page with tabs
│   └── knowledge/               # Knowledge/chat feature
└── api/                         # API routes (minimal - 2 routes)
    ├── auth/session/refresh/    # JWT refresh endpoint
    └── billing/webhook/         # Polar webhook receiver

components/
├── ui/                          # shadcn/ui components
├── layout/                      # Layout components (header, sidebar, user menu)
├── billing/                     # Billing components (plans modal, subscription status)
├── members/                     # Member management components
└── cognitive/                   # AI chat components

lib/
├── actions/                     # Server Actions
│   ├── auth/                   # Auth actions (send magic link, consume, logout)
│   └── billing/                # Billing actions (checkout, cancel, verify payment)
├── api/                        # API client and repositories
│   └── api/
│       ├── client/             # API client with auth, retry, error handling
│       └── repositories/       # Repository pattern for Go backend
├── auth/                       # Authentication utilities
│   ├── stytch/                # Stytch B2B client setup
│   ├── constants.ts           # Cookie names, routes
│   ├── server-permissions.ts  # Permission checking
│   └── token-utils.ts         # JWT utilities
├── polar/                      # Polar billing integration
│   ├── client.ts              # Polar SDK client
│   ├── subscription.ts        # Subscription fetching
│   ├── current-subscription.ts # Subscription state resolution
│   ├── plans.ts               # Plan definitions
│   └── usage.ts               # Usage metering
├── contexts/                   # React contexts
│   └── auth-context.tsx       # Auth state management
├── hooks/                      # Custom hooks
│   ├── queries/               # TanStack Query hooks
│   └── mutations/             # TanStack Mutation hooks
├── models/                     # TypeScript type definitions
├── providers/                  # Provider components
├── stores/                     # Zustand stores
└── utils/                      # Utility functions
    └── server-action-helpers.ts # ActionResult type and helpers

docs/                           # Comprehensive documentation
├── 01-getting-started.md
├── 02-authentication.md
├── 03-permissions-and-roles.md
├── 04-payments-and-billing.md
├── 05-making-api-requests.md
├── 06-creating-pages.md
├── 07-creating-apis.md
├── 08-using-hooks.md
├── 09-adding-a-feature.md
├── 10-server-actions.md
├── 11-feature-guards.md
├── 12-subscription-patterns.md
└── API-LOGGING.md
```

## Documentation

Comprehensive guides in `docs/`:
- **01-10**: Core guides (getting started, auth, permissions, billing, APIs, hooks, etc.)
- **11-feature-guards.md**: Protecting features with auth, permission, and subscription guards
- **12-subscription-patterns.md**: Managing subscriptions, checkout, and billing with Polar.sh

## Current State

- **Production-ready** authentication with Stytch B2B (magic links, sessions, RBAC)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moasq/production-saas-starter](https://github.com/moasq/production-saas-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
