---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Kredi Takip** is a Next.js 14 credit tracking application for Turkish users to manage their loans, credit cards, and financial health. The application uses Supabase for backend services, PayTR for payment processing, and Google Gemini AI for OCR analysis of bank statements.

## Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Database**: Supabase (PostgreSQL)
- **UI**: React 18, Radix UI, Tailwind CSS, shadcn/ui
- **Authentication**: Supabase Auth
- **Payment**: PayTR (migrated from Paddle)
- **AI/OCR**: Google Gemini AI
- **Email**: MailerSend/Mailjet
- **State Management**: React hooks, server components
- **Charts**: Chart.js, Recharts
- **PDF Generation**: jsPDF
- **Form Handling**: React Hook Form + Zod validation
- **Encryption**: crypto-js for sensitive data (banking credentials)

## Development Commands

```bash
# Development
pnpm dev              # Start dev server on localhost:3000
pnpm build            # Build for production
pnpm start            # Start production server
pnpm lint             # Run ESLint

# Database migrations
npx supabase db reset           # Reset local database
npx supabase migration new      # Create new migration
npx supabase db push            # Push migrations to remote
node scripts/apply-migrations.js  # Apply migrations programmatically
```

## Architecture

### Directory Structure

```
app/                    # Next.js 14 App Router pages
├── (marketing)/        # Public pages (landing, pricing, etc.)
├── admin/              # Admin dashboard
├── api/                # API routes
├── auth/               # Authentication pages
├── blog/               # Blog system
└── uygulama/           # Main application (authenticated)
    ├── ana-sayfa/      # Dashboard
    ├── krediler/       # Loans management
    ├── kredi-kartlari/ # Credit cards
    ├── hesaplar/       # Bank accounts
    ├── finansal-saglik/ # Financial health analysis
    ├── raporlar/       # Reports
    └── ayarlar/        # Settings

components/             # React components
├── ui/                 # shadcn/ui components
├── layout/             # Header, Footer
├── reports/            # Chart components
└── subscription/       # Subscription UI

lib/                    # Utilities and business logic
├── api/                # API client functions
├── supabase/           # Supabase clients
├── utils/              # Utility functions
└── types.ts            # TypeScript type definitions

supabase/
├── migrations/         # Database migration files
└── database.sql        # Full schema backup

scripts/                # Utility scripts for cron jobs
```

### Key Architectural Patterns

#### Server vs Client Components

- **Server Components** (default): Used for data fetching, authentication checks, and layouts
- **Client Components**: Used for interactivity, forms, and state management (marked with `'use client'`)
- Pages fetch data server-side, components handle UI interactions client-side

#### Authentication Flow

1. Middleware (`middleware.ts`) refreshes Supabase sessions on every request
2. Uses Supabase SSR cookies for session management
3. `createServerClient()` for server components/API routes
4. Protected routes check user session, redirect to `/giris` if unauthenticated
5. Admin routes use `lib/admin-check.ts` to verify admin role

#### Database Access Patterns

- **Server Components/API Routes**: Use `createServerClient()` from `lib/supabase/server.ts`
- **Admin Operations**: Use `createServiceRoleClient()` for elevated permissions
- **Client Side**: Use `createClient()` from `lib/supabase/client.ts`
- All database types defined in `lib/types.ts` (Database interface)

#### Subscription System

- **Plans**: Free, Pro (monthly/yearly), Premium (monthly/yearly)
- **Features**: OCR analysis credits, AI financial health analysis, ad-free experience
- **Payment**: PayTR integration (previously Paddle, fully migrated)
- **Database Tables**:
  - `subscriptions`: User subscription records
  - `subscription_plans`: Available plans
  - `invoices`: Payment invoices
  - `payment_transactions`: Payment history

#### Encryption for Sensitive Data

- Banking credentials stored encrypted using `lib/utils/encryption.ts`
- Uses AES encryption with `ENCRYPTION_KEY` environment variable
- Never expose decrypted passwords in logs or client-side code

### Important Database Tables

- **profiles**: User profile information
- **credits**: Loan/credit records with payment tracking
- **payment_plans**: Installment schedules
- **credit_cards**: Credit card tracking
- **accounts**: Bank account balances
- **banking_credentials**: Encrypted bank login credentials
- **notifications**: User notifications
- **financial_profiles**: Income, expenses, assets data
- **risk_analyses**: AI-generated financial health analysis

### API Route Patterns

```typescript
// Standard API route structure
import { createServerClient } from "@/lib/supabase/server"

export async function GET(request: Request) {
  const supabase = await createServerClient()
  const { data: { user } } = await supabase.auth.getUser()


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mittrandill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
