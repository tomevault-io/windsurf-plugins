---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

This is a **multi-event RSVP, waiver signing, and attendance tracking system** for **Fubar Labs** (New Jersey's first hackerspace) built with **Remix** and deployed to **Cloudflare Pages**. The system manages robot combat events, maker faires, workshops, and community gatherings with persistent database storage.

When working with this codebase:
- **Prioritize test-driven development (TDD)** - Write tests before implementation
- Use Git and GitHub CLI (`gh`) for all version control operations
- Ask clarifying questions before making architectural changes
- Read relevant files thoroughly before proposing changes

## Key Features

### User-Facing
- Browse upcoming events (robot combat, workshops, maker faires)
- RSVP with email confirmation
- Digital waiver signing with consent tracking
- QR code generation for event check-ins
- View personal RSVP history
- Email notifications and reminders

### Admin-Facing (Protected by Login)
- Create and manage events
- View RSVP lists and statistics
- Check attendees in (manual or QR code scan)
- Export attendance data
- Send bulk emails to attendees
- Manage waitlists and capacity

### Future Enhancements
- Discord integration for announcements
- Claude AI agent for answering event questions
- Real-time check-in dashboard (Durable Objects)
- Stripe integration for paid events
- Equipment reservation system

## Architecture Overview

### Technology Stack
- **Framework**: Remix 2.x (React Router v6)
- **UI**: React 18 + TypeScript + Tailwind CSS + shadcn/ui
- **Database**: Cloudflare D1 (SQLite) or Turso
- **ORM**: Drizzle ORM (type-safe, lightweight)
- **Deployment**: Cloudflare Pages + Workers
- **Auth**: Session-based (Cloudflare KV for sessions)
- **Email**: Resend or SendGrid
- **Package Manager**: pnpm (fast, efficient)

### Request Flow
```
User Browser
  ↓
Remix App (Cloudflare Pages)
  ↓
Loaders (Data Fetching)
  ↓
Database (D1 or Turso)
  ↓
Actions (Form Submissions)
  ↓
Server-Side Logic
  ↓
Response (SSR or JSON)
```

### Directory Structure
```
app/
├── routes/                    # File-based routing
│   ├── _index.tsx            # Homepage - upcoming events list
│   ├── events.$id.tsx        # Event detail + RSVP form
│   ├── events.$id.waiver.tsx # Waiver signing page
│   ├── events.$id.checkin.tsx # QR code check-in page
│   ├── my-rsvps.tsx          # User's RSVP history
│   ├── admin.tsx             # Admin layout with auth check
│   ├── admin._index.tsx      # Admin dashboard
│   ├── admin.events.tsx      # Event management
│   ├── admin.events.new.tsx  # Create new event
│   ├── admin.events.$id.tsx  # Edit event + attendee list
│   └── admin.login.tsx       # Admin login
├── components/
│   ├── ui/                   # shadcn/ui components
│   ├── EventCard.tsx         # Event preview card
│   ├── RSVPForm.tsx          # Reusable RSVP form
│   ├── WaiverForm.tsx        # Digital waiver component
│   ├── QRCodeDisplay.tsx     # QR code generator
│   ├── CheckInScanner.tsx    # QR scanner for check-ins
│   └── AdminNav.tsx          # Admin navigation
├── lib/
│   ├── db.server.ts          # Database connection + Drizzle setup
│   ├── schema.ts             # Drizzle schema definitions
│   ├── auth.server.ts        # Session management
│   ├── email.server.ts       # Email sending utilities
│   └── qr.server.ts          # QR code generation
├── utils/
│   ├── validation.ts         # Zod schemas for forms
│   └── date.ts               # Date formatting helpers
└── root.tsx                  # App shell, meta tags, global styles

db/
├── migrations/               # SQL migration files
└── seed.ts                  # Sample data for development

tests/
├── routes/                  # Route tests (loaders, actions)
├── components/              # Component tests
└── e2e/                     # Playwright end-to-end tests

public/
├── favicon.ico
└── robots.txt
```

### Key Architectural Patterns
- **File-Based Routing**: Routes map directly to URLs
- **Loader Pattern**: Server-side data fetching before render
- **Action Pattern**: Server-side form handling with progressive enhancement
- **Nested Layouts**: Shared UI components (admin layout, event layout)
- **Type Safety**: End-to-end TypeScript with Drizzle ORM
- **Progressive Enhancement**: Forms work without JavaScript

## Common Commands

```bash
# Setup
pnpm install                  # Install dependencies

# Development
pnpm dev                      # Start dev server (localhost:3000)
pnpm typecheck                # Run TypeScript type checking

# Database
pnpm db:generate              # Generate migrations from schema
pnpm db:migrate               # Run migrations (local)
pnpm db:migrate:prod          # Run migrations (production D1)
pnpm db:seed                  # Seed database with sample data
pnpm db:studio                # Open Drizzle Studio (database GUI)

# Testing (ALWAYS run before committing)
pnpm test                     # Run all tests (Vitest)
pnpm test:watch               # Run tests in watch mode
pnpm test:e2e                 # Run Playwright E2E tests
pnpm test:e2e:ui              # Run E2E tests with UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ricklon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
