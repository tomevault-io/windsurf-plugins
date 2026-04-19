---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Budget Reduction Tracking is a full-stack web application for tracking debt reduction and credit management progress. The system provides account management, transaction tracking, predictive analytics, and data visualization to help users achieve financial freedom.

**Tech Stack**: React 19 + TypeScript (frontend), Node.js/Express + Prisma + PostgreSQL (backend)

## Common Commands

### Backend Development
```bash
cd backend

# Development
npm run dev                  # Start dev server with hot reload (port 3001)
npm test                     # Run tests
npm run test:watch           # Run tests in watch mode
npm run test:coverage        # Generate coverage report

# Database (Prisma)
npx prisma migrate dev       # Create and apply migration
npx prisma migrate deploy    # Apply migrations (production)
npx prisma generate          # Generate Prisma Client
npx prisma studio            # Open database GUI
npx prisma migrate reset     # Reset database (dev only - destructive)

# Build
npm run build                # Build TypeScript
npm start                    # Start production server
```

### Frontend Development
```bash
cd frontend

# Development
npm run dev                  # Start Vite dev server (port 5173)
npm test                     # Run tests with Vitest
npm run test:watch           # Run tests in watch mode
npm run test:coverage        # Generate coverage report

# Build
npm run build                # Build for production
npm run preview              # Preview production build
npm run type-check           # TypeScript type checking
npm run lint                 # Run ESLint
```

### Running a Single Test
```bash
# Backend - Jest
cd backend
npm test -- auth.service.test.ts          # Specific test file
npm test -- --testNamePattern="login"     # Tests matching pattern

# Frontend - Vitest
cd frontend
npm test -- AccountList.test.tsx          # Specific test file
npm test -- --grep "should render"        # Tests matching pattern
```

## Architecture & Code Structure

### Three-Layer Architecture

**Backend** (`/backend/src/`):
```
controllers/     → Request handlers (thin, delegate to services)
services/        → Business logic (core functionality)
validators/      → Zod schemas for request validation
middleware/      → Auth, error handling, logging
routes/          → Express route definitions
```

**Frontend** (`/frontend/src/`):
```
pages/           → Top-level page components
components/      → Reusable UI components
hooks/           → Custom React hooks
services/        → API client layer (axios)
types/           → TypeScript type definitions
```

### Data Model (Prisma Schema)

**Core Entities**:
- `User` - Authentication and user management
- `Account` - Credit cards, loans, mortgages with interest tracking
- `Transaction` - Payments, charges, adjustments, interest
- `Snapshot` - Historical balance tracking for trend analysis

**Key Relationships**:
- User → Accounts (one-to-many, cascade delete)
- Account → Transactions (one-to-many, cascade delete)
- Account → Snapshots (one-to-many, cascade delete)

### Authentication Flow

JWT-based authentication with refresh tokens:
1. Login: POST `/api/auth/login` → returns access token (15min) + refresh token (7d)
2. Protected endpoints require Bearer token in Authorization header
3. Token refresh: POST `/api/auth/refresh` with refresh token
4. Auth middleware (`backend/src/middleware/auth.middleware.ts`) validates JWT
5. Frontend stores tokens via axios interceptors and React Query

### API Patterns

**Standard Response Format**:
```typescript
// Success
{ success: true, data: {...} }

// Error
{ success: false, error: { message: "...", code: "..." } }
```

**Endpoint Naming**:
- Auth: `/api/auth/login`, `/api/auth/register`
- Accounts: `/api/accounts`, `/api/accounts/:id`
- Transactions: `/api/accounts/:accountId/transactions`
- Analytics: `/api/analytics/overview`, `/api/analytics/projections`
- Health: `/api/health` (unauthenticated)

### Interest Calculations

**Daily Interest Accumulation**:
```typescript
const dailyRate = annualRate / 365;
const dailyInterest = balance * dailyRate;
```

**Payoff Projections**: Located in `backend/src/services/projection.service.ts`. Algorithm simulates monthly payments with compound interest to estimate payoff timeline and total interest cost.

### Chart Visualization

**Inverted Progress Display**: Charts show debt reduction as upward progress (decreasing balance = positive trend). Implementation uses Chart.js with inverted Y-axis or calculated reduction amounts.

Primary charts:
- Balance reduction timeline (line chart)
- Interest forecast (area chart)
- Payment distribution (pie chart)

## Database Conventions

**All IDs are UUIDs** (not integers)

**Decimal Fields**: Use Prisma's `Decimal` type for financial data (precision: 12,2)
```typescript
// Correct
currentBalance: Decimal @db.Decimal(12, 2)

// Incorrect - never use Float for money
currentBalance: Float
```

**Timestamps**: All models have `createdAt` and `updatedAt` (except Snapshot which only has `createdAt`)

**Indexes**: Critical indexes on:
- `User.email`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmcguire80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
