---
trigger: always_on
description: TeamUp (branded as "WebPadel" / "We Team Up") is a web application for padel and sports players to find partners, organize events, and build community. Users create/join events (max 4 players typical for padel), manage friends, share post-event memories, and browse venue locations. The app is focused on the Lithuanian market (Vilnius-area venues).
---

# CLAUDE.md - TeamUp (WebPadel) Codebase Guide

## Project Overview

TeamUp (branded as "WebPadel" / "We Team Up") is a web application for padel and sports players to find partners, organize events, and build community. Users create/join events (max 4 players typical for padel), manage friends, share post-event memories, and browse venue locations. The app is focused on the Lithuanian market (Vilnius-area venues).

**Live URL:** https://teamup.lt / https://weteamup.app

## Tech Stack

- **Frontend:** React 18.2 + TypeScript (strict mode), bootstrapped with Create React App via CRACO
- **Styling:** Tailwind CSS 3.4 + Material-UI 5 + custom CSS variables (shadcn/ui-style HSL theming)
- **Routing:** React Router DOM 6
- **Backend/Database:** Supabase (PostgreSQL, Auth, Storage, Realtime) — migrating from Firebase
- **Email:** SendGrid (via Vercel API route at `api/send-email.ts`)
- **Maps:** MapLibre GL / React Map GL with MapTiler geocoding
- **i18n:** i18next (minimal setup, English only currently)
- **Build tool:** CRACO (wraps react-scripts)
- **Deployment:** Vercel (static build + serverless API routes)

## Commands

```bash
npm start          # Start dev server (CRACO)
npm run build      # Production build (CI=false used in CI to ignore warnings)
npm test           # Run tests (Jest + React Testing Library)
npm run postinstall # Runs patch-package automatically after install
npm run update-events  # Run script: tsx src/scripts/updateEventsSportType.ts
npm run update-status  # Run script: tsx src/scripts/updateEventStatus.ts
```

**Node requirement:** >=18.0.0

## Project Structure

```
teamup/
├── src/
│   ├── api/              # API webhook handlers
│   ├── assets/           # Static assets (avatars, images)
│   ├── components/       # Reusable UI components (flat structure, ~40 files)
│   │   └── ui/           # Primitive UI components (button, card, input, etc. - shadcn-style)
│   ├── constants/        # Static data (locations.ts with venue coords, avatars.ts)
│   ├── contexts/         # React Context providers
│   │   ├── SupabaseAuthContext.tsx  # Supabase auth provider (NEW)
│   │   ├── AuthContext.tsx          # Legacy Firebase auth provider
│   │   ├── CookieContext.tsx
│   │   └── ThemeContext.tsx
│   ├── hooks/
│   │   ├── useSupabaseAuth.ts       # Supabase auth hook (NEW)
│   │   ├── useSupabaseEvents.ts     # Supabase events hook (NEW)
│   │   ├── useAuth.ts              # Legacy Firebase auth hook
│   │   ├── useEvents.ts            # Legacy Firebase events hook
│   │   └── ...
│   ├── lib/
│   │   ├── supabase.ts             # Supabase client init (NEW)
│   │   └── utils.ts                # cn() helper
│   ├── pages/            # Route-level page components (~12 pages)
│   ├── services/
│   │   ├── supabaseNotificationService.ts  # Supabase notifications (NEW)
│   │   ├── supabaseEmailService.ts         # Supabase email service (NEW)
│   │   └── ...                             # Legacy Firebase services
│   ├── types/
│   │   ├── supabase.ts             # Supabase DB types + aliases (NEW)
│   │   └── index.ts                # Legacy app types
│   ├── App.tsx           # Root component with routing
│   ├── firebase.ts       # Legacy Firebase client init
│   └── index.tsx         # Entry point
├── api/                  # Vercel serverless API routes (NEW)
│   └── send-email.ts     # SendGrid email endpoint
├── supabase/             # Supabase project config (NEW)
│   ├── config.toml
│   └── migrations/
│       └── 20250223000001_initial_schema.sql
├── functions/            # Legacy Firebase Cloud Functions
├── patches/              # patch-package patches for react/react-dom 18.2.0
├── public/               # Static public files
├── vercel.json           # Vercel deployment config (updated for API routes)
└── .github/workflows/    # CI/CD
```

## Migration Status: Firebase → Supabase

The project is migrating from Firebase to Supabase. Both systems coexist during the transition.

### What has been migrated
- **Database schema:** Full PostgreSQL migration in `supabase/migrations/` with RLS policies
- **Auth hooks:** `useSupabaseAuth.ts` replaces `useAuth.ts`
- **Auth context:** `SupabaseAuthContext.tsx` replaces `AuthContext.tsx`
- **Events hook:** `useSupabaseEvents.ts` replaces `useEvents.ts`
- **Notifications:** `supabaseNotificationService.ts` with Realtime subscriptions
- **Email service:** `supabaseEmailService.ts` + Vercel API route (`api/send-email.ts`)
- **Storage:** 3 Supabase storage buckets (avatars, event-covers, memory-images)
- **Types:** Full Supabase DB types in `src/types/supabase.ts`

### What still uses Firebase (to migrate)
- Page components directly importing from `firebase/firestore` (EventDetails, Home, Community, etc.)
- Component-level Firestore calls (CreateEventDialog, EditEventDialog, etc.)
- Firebase Storage upload calls in components
- `firebase.ts` and `firebaseAdmin.ts` client init files

### Migration pattern for remaining components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dzekuza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
