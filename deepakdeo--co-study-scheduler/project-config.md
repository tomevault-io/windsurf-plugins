---
trigger: always_on
description: A multi-tenant real-time scheduling tool where anyone can create a "room," get a shareable link, and let others book 2-hour co-study sessions. All times auto-convert to the viewer's local timezone. The host shares the link with their study group; members open it, see available slots in their own timezone, and book.
---

# Co-Study Scheduler

## Project Overview
A multi-tenant real-time scheduling tool where anyone can create a "room," get a shareable link, and let others book 2-hour co-study sessions. All times auto-convert to the viewer's local timezone. The host shares the link with their study group; members open it, see available slots in their own timezone, and book.

## Tech Stack
- **Frontend:** Vite + React + Tailwind CSS
- **Database/Backend:** Supabase (PostgreSQL + Realtime + Edge Functions + RLS)
- **Hosting:** Vercel (static build deployment)
- **Email:** Resend (transactional emails via Supabase Edge Functions)
- **Routing:** react-router-dom v6+
- **Timezone handling:** date-fns + date-fns-tz (NOT moment.js)
- **Language:** JavaScript (not TypeScript for v1 — keep it simple)

## Pages & Routes

### 1. Landing Page (`/`)
- Hero section explaining what the tool does
- "Create a Room" CTA button
- Brief "How it works" section (3 steps: create room → share link → study together)

### 2. Create Room (`/create`)
- Form fields:
  - Host name (required)
  - Room title (required, e.g., "AI/ML Co-Study with Deo")
  - Description (optional)
  - Slug (auto-generated from title, editable, validated for uniqueness)
  - Zoom/meeting link (required — private, never shown on public board, delivered only via email/confirmation)
  - Host timezone (auto-detected, editable dropdown)
  - Morning/afternoon window: start hour, end hour (defaults: 10 AM – 3 PM)
  - Evening window: start hour, end hour (defaults: 7 PM – 11 PM)
  - Slot duration: fixed at 120 minutes for v1
  - Slot interval: 30 minutes (rolling start times)
  - Admin PIN (required, 4-6 digits, for accessing admin view)
- On submit: create room in Supabase, redirect to `/r/:slug`

### 3. Public Room View (`/r/:slug`)
- Room header: title, host name, description, session format reminder
- Timezone indicator: auto-detected local TZ with option to change
- Week navigator: prev/next week arrows, current week range display
- Slot grid:
  - Columns = Mon–Fri (with dates)
  - Rows = available 2-hour slots at 30-min intervals within configured windows
  - Morning/afternoon and evening sections visually separated
  - Cell states: Open (green), Booked (orange/amber, shows booker's name), Blocked/overlap (gray), Past (dimmed)
- Click open slot → booking form appears
- Booking form (3 fields):
  - Name (required)
  - Email (optional — helper text: "Optional — to receive your Zoom link by email")
  - Private note to host (optional — helper text: "Only the host sees this")
- After booking → confirmation screen:
  - Shows date/time in booker's timezone
  - If email provided: "You're booked! You'll receive a Zoom meeting link at [email] before the session."
  - If no email: "You're booked! Deo will share your Zoom meeting link via direct message in the AI/ML Career Launchpad group."
  - Session format reminder: 5–10 min hello & goals → ~100 min focused study → 10–15 min wrap-up & chat
- Real-time updates: when someone else books while you're viewing, the board updates live via Supabase Realtime

### 4. Admin View (`/r/:slug/admin`)
- Protected by admin PIN (simple form, stored in room record)
- Shows everything on public board PLUS:
  - Booker's email (if provided)
  - Private note (if provided)
  - Cancel booking button per slot
  - Blackout: ability to mark specific dates as unavailable
- Summary section: total bookings this week, upcoming next session

## Database Schema

All timestamps stored in UTC. Host timezone stored as IANA string (e.g., `America/Chicago`).

```sql
-- Rooms table
create table rooms (
  id uuid default gen_random_uuid() primary key,
  slug text unique not null,
  host_name text not null,
  title text not null,
  description text,
  meeting_link text not null,
  host_timezone text default 'America/Chicago',
  morning_start int default 10,
  morning_end int default 15,
  evening_start int default 19,
  evening_end int default 23,
  slot_duration int default 120,
  slot_interval int default 30,
  admin_pin text not null,
  is_active boolean default true,
  created_at timestamptz default now()
);

-- Bookings table
create table bookings (
  id uuid default gen_random_uuid() primary key,
  room_id uuid references rooms(id) on delete cascade,
  name text not null,
  email text,
  note text,
  booking_date date not null,
  slot_start_utc timestamptz not null,
  slot_end_utc timestamptz not null,
  booker_timezone text,
  status text default 'confirmed' check (status in ('confirmed', 'cancelled')),
  created_at timestamptz default now()
);

-- Indexes
create index idx_bookings_room_date on bookings(room_id, booking_date);
create index idx_rooms_slug on rooms(slug);

-- Row Level Security
alter table rooms enable row level security;
alter table bookings enable row level security;

-- Public read access for rooms
create policy "Rooms are publicly readable"
  on rooms for select using (is_active = true);

-- Public read access for confirmed bookings
create policy "Confirmed bookings are publicly readable"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepakdeo/co-study-scheduler](https://github.com/deepakdeo/co-study-scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
