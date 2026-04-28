---
trigger: always_on
description: Composer is a date and night-out itinerary generator for NYC. Users answer a short cascading questionnaire (occasion → neighborhood → budget → vibe) and receive a curated 3-stop evening itinerary structured as **Opener → Main → Closer**.
---

# CLAUDE.md — Composer

## Project Overview

Composer is a date and night-out itinerary generator for NYC. Users answer a short cascading questionnaire (occasion → neighborhood → budget → vibe) and receive a curated 3-stop evening itinerary structured as **Opener → Main → Closer**.

The product is built on a **hybrid curation model**: the venue database is human-curated by the founders (Reid and Adit), scored and assembled by weighted algorithm, and polished by the Claude API for copy voice. The human taste layer is the core differentiator — this is not a generic AI recommendation engine.

**Primary target: Mobile-responsive web.** Website first at onpalate.com/composer. iOS via Capacitor is Phase 2. Every UI decision should work on a phone screen first.

**Auth: Supabase email/password.** Users sign in or sign up on a single combined entry screen (`AuthScreen`) with email + password — the form tries `signInWithPassword` first and falls back to `signUp` on an "invalid / not found" error, so there's no explicit sign-in/sign-up toggle. Forgot-password flow uses `resetPasswordForEmail` with a redirect to `/auth/reset`. No OAuth providers. Profile and saved itineraries live in Supabase tables with RLS (`composer_users`, `composer_saved_itineraries`). The one exception to "no client persistence" is the page-to-page sessionStorage bridge between `/compose` and `/itinerary` — that's in-tab flight state, not user state.

---

## Tech Stack

- **Framework**: Next.js 14+ (App Router) + TypeScript
- **Styling**: Tailwind CSS
- **Database**: Supabase (PostgreSQL + Row Level Security)
- **AI**: Google Gemini 2.5 Flash (`gemini-2.5-flash`) — itinerary copy and voice
- **Weather**: OpenWeatherMap API — called per generation, not cached
- **Package Manager**: npm
- **Deployment**: Vercel
- **Mobile (Phase 2)**: Capacitor → iOS

---

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
GEMINI_API_KEY
OPENWEATHERMAP_API_KEY
```

Never hardcode these. Never commit `.env.local`. Always use `process.env.*` server-side and `NEXT_PUBLIC_*` only when the value is safe to expose to the client.

---

## Project Structure

```
app/
├── page.tsx                  # Landing page
├── compose/
│   └── page.tsx              # Questionnaire flow
├── itinerary/
│   └── page.tsx              # Composition output
└── api/
    └── generate/
        └── route.ts          # POST endpoint: weather + scoring + Claude → itinerary

components/
├── ui/                       # Base UI primitives (Button, OptionCard, ProgressBar)
├── providers/                # AuthProvider (context for user/profile/session)
├── landing/                  # Hero, CTA
├── home/                     # HomeScreen (signed-in landing with saved plans)
├── auth/                     # AuthScreen + ForgotPasswordScreen (email/password)
├── onboarding/               # OnboardingFlow (name + profile, session already active)
├── questionnaire/            # QuestionnaireShell, StepLoading, OptionCard
└── itinerary/                # CompositionHeader, StopCard, WalkConnector, ActionBar

lib/
├── supabase.ts               # Anon Supabase client for non-auth reads (venues)
├── supabase/browser.ts       # Browser auth-aware client (@supabase/ssr, cookie session)
├── supabase/server.ts        # Server auth-aware client for Route Handlers
├── auth.ts                   # Sign in / sign up / reset-password / profile helpers
├── scoring.ts                # Weighted venue scoring + itinerary composer
├── weather.ts                # OpenWeatherMap fetch + rain/snow classification
├── geo.ts                    # Haversine distance + Manhattan grid correction + Maps URL builder
├── claude.ts                 # Gemini API call + graceful fallback
└── sharing.ts                # URL param encode/decode for share links

config/
├── options.ts                # All questionnaire step definitions
└── prompts.ts                # Claude system prompt + generation prompt builder

supabase/
└── seed.sql                  # Schema + seed venues
```

---

## Database Schema

```sql
-- See supabase/migrations/20260419_venue_schema_v2.sql for the full DDL.
composer_venues (
  id uuid primary key,
  name text not null, neighborhood text not null, category text not null,
  price_tier int check (1-4),
  vibe_tags text[], occasion_tags text[], stop_roles text[],
  duration_hours int, outdoor_seating text ('yes'/'no'/'unknown'),
  reservation_difficulty int, reservation_url text, maps_url text,
  curation_note text, awards text, curated_by text, signature_order text,
  address text, latitude float not null, longitude float not null,
  active boolean, notes text, hours text, last_verified date,
  happy_hour text, dog_friendly boolean, kid_friendly boolean,
  wheelchair_accessible boolean, cash_only boolean,
  quality_score int default 7, curation_boost int default 0,
  created_at timestamptz
)
```

### Canonical Vibe Tags

Tag lists are generated from the Google Sheet via `npm run generate-configs`.
To add a tag: edit the sheet's Vibe Tags or Vibe Scoring Matrix tab, re-run the script.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryaadit/composer](https://github.com/aryaadit/composer) — distributed by [TomeVault](https://tomevault.io/claim/aryaadit).
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
