---
trigger: always_on
description: This is a basketball pickup game ("runs") organizer web app. A "run" is basketball slang for a scheduled pickup game session. The app allows users to RSVP for runs and admins to manage the community.
---

# Zach's Runs - AI Assistant Rules

## Project Overview
This is a basketball pickup game ("runs") organizer web app. A "run" is basketball slang for a scheduled pickup game session. The app allows users to RSVP for runs and admins to manage the community.

## Tech Stack
- **Frontend**: Next.js 16 with TypeScript, Tailwind CSS, App Router
- **Backend**: Flask (Python) with SQLAlchemy ORM
- **Database**: SQLite (local) / Neon Postgres (production via Vercel)
- **Deployment**: Vercel (frontend + serverless Python backend)
- **Email**: Resend API with QStash queue (Upstash)

## Email System Architecture

### How It Works
- **Local Development**: In-memory queue with background daemon thread (1 email/sec rate limit)
- **Production**: QStash (Upstash) handles scheduling and delivery to worker endpoint
- **Fire-and-Forget**: API returns immediately; emails processed async

### Email Flow
```
Local:      API → Queue Email → Return Success → Background Thread (1/sec)
Production: API → Enqueue to QStash → Return Success → QStash → /api/email-worker
```

### Email Types Implemented
1. Welcome Email (on signup - sent even if not verified)
2. Account Verified Email (when admin verifies user)
3. Admin New User Notification (to all admins on signup)
4. Run Created Email (to verified users)
5. Run Reminder Email (manual admin trigger)
6. Run Completed Email (to attendees)
7. Run Cancelled Email (when run deleted)
8. Run Modified Email (when run updated, shows what changed)
9. Announcement Email (to verified users)

### Key Files
- `backend/utils/email.py` - Email utilities, queue system, send functions
- `backend/routes/email_worker.py` - QStash worker endpoint (`/api/email-worker`)
- `backend/templates/emails/` - Jinja2 HTML email templates

### Environment Variables for Email
```bash
RESEND_API_KEY=xxx                    # Resend API key
EMAIL_FROM_ADDRESS=notifications@send.zachsruns.com
EMAIL_FROM_NAME="Zach's Organized Runs"
FRONTEND_URL=https://zachsruns.vercel.app
QSTASH_TOKEN=xxx                      # Production only
QSTASH_CURRENT_SIGNING_KEY=xxx        # Production only
QSTASH_NEXT_SIGNING_KEY=xxx           # Production only
LOCAL_USE_QSTASH=true                 # Optional: test QStash locally (requires ngrok)
QSTASH_CALLBACK_URL=xxx               # Your ngrok URL for local QStash testing
```

### Testing QStash Locally with ngrok

The local threading system works for most cases, but to test the actual QStash queue implementation (signature verification, scheduling, retries), use ngrok:

**Terminal 1 - Start ngrok:**
```bash
ngrok http 5001
# Copy the HTTPS URL (e.g., https://abc123.ngrok-free.app)
# ⚠️ FREE NGROK: URL changes every restart! Update .env each time.
```

**Terminal 2 - Backend with QStash:**
```bash
cd backend
source venv/bin/activate
# Edit .env with the NEW ngrok URL from Terminal 1:
#   LOCAL_USE_QSTASH=true
#   QSTASH_CALLBACK_URL=https://abc123.ngrok-free.app  ← paste new URL here
python app.py
# ⚠️ Must restart backend if ngrok URL changes
```

**Terminal 3 - Frontend:**
```bash
cd frontend
npm run dev
```

**How it works:**
1. Backend enqueues emails to QStash (external service)
2. QStash calls back to your ngrok URL → local backend `/api/email-worker`
3. Worker verifies signature and sends email via Resend
4. Email links point to `localhost:3000` (via `FRONTEND_URL`)

**When to use:** Testing QStash-specific behavior (signature verification, scheduled delays, retry logic). For normal development, the default threading system is simpler.

### Email Conventions
- Only verified users receive emails (exception: welcome email on signup)
- Templates extend `base.html` with basketball theme styling
- All run emails include "View Runs →" link to main site
- Local dev filters emails to real addresses only (`zmann`, `rantesting22`)

## Project Structure
```
ZachsRuns/
├── frontend/           # Next.js app
│   ├── app/           # App Router pages
│   ├── components/    # React components
│   ├── context/       # React contexts (AuthContext)
│   ├── lib/           # API client, auth utilities
│   └── types/         # TypeScript interfaces
├── backend/           # Flask API
│   ├── routes/        # Blueprint route handlers
│   ├── models.py      # SQLAlchemy models
│   ├── database.py    # DB initialization
│   ├── middleware.py  # Auth middleware
│   └── utils/         # Email utilities
└── frontend/api/      # Vercel serverless entry point
```

## Code Style & Conventions

### Frontend (TypeScript/React)
- Use `'use client'` directive for components with hooks/interactivity
- Import types from `@/types` (path alias configured)
- Use `@/lib/api` functions for all API calls (never raw fetch)
- Use `@/context/AuthContext` for auth state via `useAuth()` hook
- Tailwind classes for styling - use custom colors from tailwind.config.js
- Component files are PascalCase (e.g., `RunCard.tsx`)
- Page files use Next.js App Router convention (`page.tsx`)

### Custom Tailwind Colors
```js
'basketball-orange': '#FF6B35'
'basketball-black': '#1A1A1A'
'wood-light': '#D4A574'
'wood-medium': '#8B6F47'
'wood-dark': '#5C4A37'
```

### Backend (Python/Flask)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZachManno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
