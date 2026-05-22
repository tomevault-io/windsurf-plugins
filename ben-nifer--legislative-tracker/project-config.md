---
trigger: always_on
description: > This file provides context to Claude Code CLI (running in VS Code integrated terminal) about our project. Keep it updated as the project evolves.
---

# CLAUDE.md - Legislative Tracker

> This file provides context to Claude Code CLI (running in VS Code integrated terminal) about our project. Keep it updated as the project evolves.

## Project Overview

**Legislative Tracker** is a Letterboxd-style civic engagement platform for New York City Council legislation. Users can browse, track, review, and engage with NYC legislation in a social, accessible way.

**GitHub Repository:** `legislative-tracker`
**Development Environment:** Claude Code CLI in VS Code integrated terminal

### Core Value Proposition
"Make NYC legislation accessible and engaging for everyday New Yorkers"

### Target Users
- NYC residents who want to stay informed
- Advocacy groups tracking specific issues
- Journalists covering city politics
- Anyone curious about how NYC laws are made

---

## MVP Features (Version 1)

### Home Page
- **Personal tracked legislation overview** (click to expand to full watchlist)
- **Trending legislation section** (click to expand to full trending page)
- **"New from friends/followed accounts"** feed (activity from followed users and council members)

### Search
- **Legislation search:**
  - Search by keyword
  - Filter by: Status, Topic, Sponsor, Date range, Trending
  - Filter by Resolution/Introduction type
  - ❌ Location filter (post-MVP)
- **Profile search:**
  - Search users by name

### Watchlist Page
- Accessible via tab or from home page
- **Widgetized hearing schedule** (expandable) for followed legislation
- **Notification toggle** per item (in-app only for MVP)

### Trending Page
- Full trending legislation list
- Filters: Status, Topic, Sponsor, Date range, Resolution/Introduction

### Legislation Display (Three Levels)

**Level 1 - Card/Overview (in lists):**
- ID#, status, summary
- Engagement counts: # support, # opposed, # neutral, # saves, # comments
- Save/bookmark button

**Level 2 - Expanded Dropdown:**
- Everything from Level 1, plus:
- Committee name, date introduced, last action (with date)
- "View Details" button

**Level 3 - Full Detail Page:**
- Complete summary (Claude API generated or official)
- Sponsor information with link to profile
- Latest updates / next scheduled hearing
- **Engagement section:**
  - Support / Oppose / Neutral counts (separate from "Watching")
  - Comments with filtering:
    - Latest
    - Most engaged (overall) - based on upvote + downvote count
    - Most engaged by stance (support/oppose/neutral commenters)
    - People you follow
  - Upvote/downvote on comments (Reddit-style, affects ranking)
- Full bill text (embedded or linked, check API availability)

### User Profile

**Public Profile Page:**
- Profile picture
- Bio
- External links (social media style - platform + URL)
- Interest tags (custom + predefined, separate from legislation topics)
- Public bookmarks
- Stance summary (Supporting/Opposing/Neutral counts)
- Following count (users + council members)
- Followers count
- Comment history (filterable by most engaged)

**Profile Settings:**
- Edit profile picture, bio, links
- Manage interest tags (select predefined or create custom)
- ❌ Private profile toggle (premium, post-MVP)

### Notifications (In-App Only for MVP)
- Updates to bills you follow
- Hearing alerts
- Bill amendments
- Engagement on your comments (replies, upvotes)

### Features NOT in MVP
- ❌ Location-based filtering
- ❌ Private profiles (premium feature)
- ❌ Interest groups
- ❌ User-created lists (Letterboxd-style)
- ❌ Email notifications
- ❌ Push notifications

---

## Tech Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| **Framework** | Next.js 14 (App Router) | Using `src/` directory, Server Components by default |
| **Language** | TypeScript | Strict mode enabled |
| **Styling** | Tailwind CSS + shadcn/ui | Dark theme, custom color palette |
| **Database** | Supabase (PostgreSQL) | Hosted, with Row Level Security |
| **Auth** | Supabase Auth | Email magic links + Google OAuth |
| **AI** | Anthropic Claude API | For legislation summaries |
| **Moderation** | OpenAI Moderation API | Free, for content safety |
| **Hosting** | Vercel | Auto-deploy from main branch |

---

## Project Structure

```
legislative-tracker/
├── CLAUDE.md                     # This file - context for Claude Code
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (auth)/               # Auth-required routes
│   │   │   ├── watchlist/        # User's watchlist page
│   │   │   ├── profile/          # User's own profile settings
│   │   │   ├── notifications/    # In-app notifications
│   │   │   └── settings/         # App settings
│   │   ├── (public)/             # Public routes
│   │   │   ├── legislation/
│   │   │   │   ├── page.tsx      # Browse/search all legislation
│   │   │   │   └── [slug]/       # Detail view (Level 3)
│   │   │   ├── trending/         # Trending page
│   │   │   ├── council-members/
│   │   │   │   ├── page.tsx      # Browse council members
│   │   │   │   └── [slug]/       # Council member profile
│   │   │   ├── users/
│   │   │   │   └── [username]/   # Public user profile
│   │   │   └── search/           # Search results page
│   │   ├── api/                  # API routes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ben-nifer/legislative-tracker](https://github.com/Ben-nifer/legislative-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
