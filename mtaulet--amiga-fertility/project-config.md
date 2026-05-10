---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Amiga Fertility is a HIPAA-compliant patient portal platform for fertility clinics. The application handles sensitive Protected Health Information (PHI) and must maintain strict security and compliance standards.

**Tech Stack:**
- Next.js 16 (App Router) with React 19 and TypeScript
- Auth0 for authentication (HIPAA-eligible tier)
- Supabase (PostgreSQL) for database with Row Level Security
- Tailwind CSS for styling

## Development Commands

```bash
npm run dev          # Start development server on localhost:3000
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
```

## Architecture

### Authentication Flow (Auth0)

- **API Routes**: `/src/app/api/auth/[auth0]/route.ts` - Handles all Auth0 endpoints
- **User Provider**: `/src/components/auth/UserProvider.tsx` - Client-side auth context
- **Middleware**: `/src/middleware.ts` - Route protection for authenticated pages
- **Protected Routes**: `/dashboard`, `/profile` require authentication

Auth0 endpoints automatically available:
- `/api/auth/login` - Initiate login
- `/api/auth/logout` - Log out user
- `/api/auth/callback` - OAuth callback
- `/api/auth/me` - Get current user

### Database Architecture (Supabase)

**Client Configuration:**
- `/src/lib/supabase/client.ts` - Client-side Supabase client (uses anon key)
- `/src/lib/supabase/server.ts` - Server-side client (uses service role key for admin operations)
- `/src/lib/supabase/types.ts` - TypeScript types matching database schema

**Schema** (`supabase/migrations/001_initial_schema.sql`):
- `patients` - Core patient information (linked to Auth0 via auth0_id)
- `patient_profiles` - Medical history, medications, allergies (JSONB fields)
- `appointments` - Clinic appointments
- `documents` - Patient documents with file URLs
- `messages` - Communication between patients and clinics

**Security:**
- All tables have Row Level Security (RLS) enabled
- Patients can only access their own data (enforced by RLS policies)
- Auth0 ID is used to identify the current user in RLS policies

### Routing Structure (Next.js App Router)

```
app/
├── api/auth/[auth0]/    # Auth0 endpoints
├── dashboard/           # Patient dashboard (protected)
├── login/              # Login page
├── profile/            # Patient profile (protected)
├── layout.tsx          # Root layout with UserProvider
└── page.tsx            # Public home page
```

### Page Patterns

**Server Components** (default):
- Use `getSession()` from `@auth0/nextjs-auth0` to check authentication
- Redirect to `/login` if not authenticated
- Can directly query database with Supabase server client

**Client Components** (use `'use client'`):
- Use `useUser()` hook for authentication state
- Use Supabase client for database queries

### Styling

- Tailwind CSS with custom primary color palette (blues)
- Responsive design with mobile-first approach
- Custom color system in `tailwind.config.js`:
  - `primary-*` colors for brand (50-900 shades)
  - Uses Tailwind's default grays and utilities

## Important Security Considerations

### HIPAA Compliance Requirements

1. **Never log or expose PHI** in console, errors, or analytics
2. **Always use HTTPS** in production (enforced by hosting)
3. **Validate all user inputs** to prevent injection attacks
4. **Use parameterized queries** (Supabase client handles this)
5. **Implement audit logging** for PHI access (to be added)

### Code Security Rules

**DO:**
- Use Supabase RLS policies for data access control
- Validate user inputs on both client and server
- Use environment variables for secrets (never commit `.env.local`)
- Use the server-side Supabase client for privileged operations
- Check authentication before accessing patient data

**DON'T:**
- Never bypass RLS policies
- Never expose service role keys to client
- Don't store PHI in localStorage or client-side storage
- Don't log patient data to console or error tracking
- Don't use `auth0_id` from user input (always get from session)

### Adding New Features

When adding features that handle patient data:

1. **Update database schema**: Add migration in `supabase/migrations/`
2. **Add TypeScript types**: Update `/src/lib/supabase/types.ts`
3. **Configure RLS policies**: Ensure patients can only access their own data
4. **Test access control**: Verify users cannot access other patients' data
5. **Add audit logging**: Log access to sensitive data (when implemented)

### Working with Supabase

**Querying Data:**
```typescript
// Client-side (in 'use client' components)
import { supabase } from '@/lib/supabase/client'
const { data, error } = await supabase.from('patients').select('*')

// Server-side (in Server Components or API routes)
import { supabaseAdmin } from '@/lib/supabase/server'
const { data, error } = await supabaseAdmin.from('patients').select('*')
```

**RLS Context:**
Server-side queries bypass RLS unless you set the context:
```typescript
// Set Auth0 ID for RLS (when not using service role)
await supabase.rpc('set_config', {
  name: 'app.current_user_auth0_id',
  value: user.sub
})
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtaulet/Amiga-Fertility](https://github.com/mtaulet/Amiga-Fertility) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
