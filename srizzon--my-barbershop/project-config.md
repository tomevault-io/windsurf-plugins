---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Angular 19 SaaS application for barbershop wait time management, built with Supabase backend. The project solves a specific problem for solo barbers who don't use appointment systems.

### Business Problem & Solution
**The Problem**: Solo barbers working without appointments need a way to communicate wait times to customers, reducing uncertainty and improving customer experience.

**The Solution**: 
- **For Barbers**: An admin dashboard to easily adjust wait times by adding/removing minutes as needed
- **For Customers**: A public storefront page to check if the barbershop is open and see real-time estimated wait times
- **Real-time Updates**: All changes are instantly reflected to customers viewing the storefront

This creates transparency for customers and helps barbers better organize their day by managing customer flow.

## Architecture

### Project Structure
- **Nx Monorepo**: Uses Nx workspace with single Angular application (`apps/my-barbershop`)
- **Domain-Driven Design**: Code is organized by business domains:
  - `auth/`: User authentication and authorization for barbers
  - `dashboard/`: Admin panel where barbers can adjust wait times (add/remove minutes)
  - `storefront/`: Public customer-facing page showing current wait times and open/closed status
  - `subscription/`: Payment and subscription management for the SaaS model
- **Shared Resources**: 
  - `shared/`: Core services, APIs, and utilities
  - `widget/`: Reusable UI components and directives
  - `core/`: Guards, layouts, and core pages

### Key Technologies
- **Frontend**: Angular 19 with standalone components
- **UI Library**: NG-Zorro (Ant Design for Angular)
- **Backend**: Supabase (Auth, Database, Real-time, Storage)
- **Payments**: Stripe integration
- **State Management**: RxJS with services pattern
- **Styling**: SCSS + Less for theming
- **Testing**: Jest for unit tests, Playwright for e2e

### Authentication & Routing
- **Auth Guard**: `core/guards/auth/auth.guard.ts` protects authenticated routes
- **Layout System**: 
  - `AuthLayout` for login/signup pages
  - `ShellLayout` for authenticated app shell
- **Route Structure**:
  - `/auth/*` - Authentication flows for barbershop owners
  - `/subscription` - Payment and setup for new barbershop subscriptions
  - `/dashboard` - Admin panel for barbers to manage wait times (auth required)
  - `/view/*` - Public storefront pages where customers check wait times (no auth required)

### Data Layer
- **Supabase Integration**: `shared/services/supabase/supabase.service.ts`
- **API Services**: Domain-specific APIs in `*/apis/` directories
- **Real-time**: Supabase real-time subscriptions for instant wait time updates from dashboard to storefront
- **Storage**: File upload handling via Supabase Storage for barbershop branding/images

## Development Commands

### Core Commands
```bash
# Start development server (port 4244)
npm start
# or
nx serve my-barbershop

# Build for production
nx build my-barbershop

# Run tests
nx test my-barbershop

# Run e2e tests
nx e2e my-barbershop-e2e

# Lint code
nx lint my-barbershop
```

### Supabase Local Development
```bash
# Start local Supabase (requires Docker)
supabase start

# Stop local Supabase
supabase stop

# Reset database
supabase db reset
```

## Configuration

### Environment Setup
1. Configure Supabase credentials in `environments/environment.ts`:
   ```typescript
   export const environment = {
     SUPABASE_URL: 'your-supabase-url',
     SUPABASE_KEY: 'your-supabase-anon-key',
   };
   ```

2. Local Supabase runs on:
   - API: http://localhost:64321
   - Studio: http://localhost:64323
   - Storage: http://localhost:64324

### Project Configuration
- **Component Prefix**: `mb-` (My Barbershop)
- **Bundle Budgets**: 500KB warning, 1MB error for initial bundle
- **Dev Server Port**: 4244
- **Theming**: Dynamic theme switching (default/dark) via Less files

## Development Patterns

### Component Architecture
- **Standalone Components**: All components use Angular's standalone API
- **Dynamic Forms**: Reusable form system in `widget/components/dynamic-form/`
- **UI Components**: Leverage NG-Zorro components consistently
- **Services**: Injectable services for data management and business logic

### State Management
- **Service-based**: Use Angular services with RxJS for state management
- **Real-time Updates**: Supabase subscriptions for live data
- **Form Handling**: Reactive forms with custom validation

### File Organization
- **Feature Modules**: Each domain has its own routing and components
- **Shared Utilities**: Common functions in `shared/utils/`
- **Type Safety**: Interfaces defined per domain in `interfaces/`
- **Constants**: Configuration objects in `constants/` directories

### Styling Conventions
- **SCSS**: Primary styling language
- **Less Theming**: Theme-specific styles for light/dark modes
- **NG-Zorro**: Consistent use of Ant Design components
- **Responsive**: Mobile-first responsive design patterns

## Core Features & User Flows

### Barber (Admin) Flow
1. **Authentication**: Login via `/auth/login` 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srizzon/my-barbershop](https://github.com/srizzon/my-barbershop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
