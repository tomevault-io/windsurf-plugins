---
trigger: always_on
description: **Title:** MyCampus — A Verified Campus + Cross-University Collaboration Platform
---

# MyCampus — Project Overview & Architecture Guide

## Project Overview

**Title:** MyCampus — A Verified Campus + Cross-University Collaboration Platform
**Institution:** C. V. Raman Polytechnic

---

## The Problem

Campus information today is scattered — spread across notices, WhatsApp groups, faculty communication, and word of mouth. Students struggle to find mentors, collaborators, research opportunities, and startups that match their interests. Results, exams, events, hackathons, and hiring updates are all fragmented across different channels.

**MyCampus's answer:** One platform. A verified campus layer, combined with connected opportunities across universities.

---

## Project Goals (4 Objectives)

1. **Unify Campus Life** — Centralize notices, events, results, exams, notes, faculty info, and general campus information in one trusted platform.
2. **Enable Discovery** — Help students find peers, mentors, projects, research, and startups aligned with their interests.
3. **Support Careers** — Surface hiring updates, hackathons, and competitions, plus structured career guidance — without ranking students.
4. **Keep It Trusted** — Keep college information private to verified users, while letting selected projects/research/startups connect across universities.

---

## Core Features (Grouped into 4 Categories)

### 1. Discovery, Mentorship & Campus
- Find similar students (peers with related interests/goals)
- Mentor discovery (seniors and experienced people)
- Collaboration matching (complementary skills for projects/research/startups)
- Semantic search (natural-language search for opportunities)
- **Interactive 2D campus map** (clickable buildings, roads, paths, with extruded depth)

### 2. Career, Research & Startup Growth
- Hiring updates (which companies are recruiting)
- Job requirements (skills/qualifications employers want)
- Faculty directory (professors + contact info)
- Research discovery (cross-university research opportunities)
- Career guidance (structured career direction)
- Startup discovery (student startups and roles they need)

### 3. Campus Life & Collaboration
- Event notifications (college events, activities, programs)
- Project discovery (student projects across universities)
- Cross-university collaboration (connect with teams beyond your own campus)
- Hackathons & competitions updates

### 4. Academics, Safety & Access
- Evacuation alerts (real-time emergency safety notifications)
- Results announcements (instant updates when results are out)
- Notes section (shared, organized study notes by subject)
- Exam dates & timetable (centralized exam schedule)
- College-verified access (notices, results, exams, safety info stay within the verified campus layer)

---

## Scope — Why MyCampus is Different

- **Not a generic social network** — combines a trusted digital campus with a wider collaboration network.
- **Campus Layer:** Students register with their official college identity to access college-specific info (notices, results, exams, safety updates).
- **Open Collaboration:** People/teams state what they're doing and need; relevant students discover and connect.
- **AI usage is targeted** — used specifically for semantic discovery and requirement matching, not as a gimmick; core campus functions run on standard software.
- **Cross-university visibility** — projects, research, startups, mentors, and opportunities are discoverable across universities according to visibility rules.

---

## Architecture Notes (Living Document)

### Current State (Admin Panel - `admin-page/`)
- React + Vite + React Router
- Tailwind CSS (custom admin design system)
- 5 admin pages: Dashboard, Notices & Events, Exams & Results, Placements, Settings
- 10 reusable components (DataTable, FilterBar, FormModal, PageLayout, StatBar, Tabs, Table, Form, Badge)
- Column definitions for 9 entity types
- `useAdminTable` hook for table state management
- Mock data files (admin, events, exams, notices, placements, universities)

### Tech Stack Decisions
| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | React 18, Vite | Admin panel complete |
| Styling | Tailwind CSS | Custom admin design tokens |
| Routing | React Router v6 | Nested routes with layout |
| Icons | Lucide React | Consistent icon system |
| State | React hooks + Context | No Redux/Zustand yet |
| Build | Vite | Fast dev + optimized build |

### Upcoming Architecture Decisions (To Be Finalized)
- [ ] **Student App** (`mycampus/`) — separate React app or shared monorepo?
- [ ] **Backend** — Node/Express, Python/FastAPI, or Supabase/Firebase?
- [ ] **Database** — PostgreSQL, MongoDB, or SQLite for dev?
- [ ] **Auth** — JWT, NextAuth, Clerk, or custom?
- [ ] **Real-time** — WebSockets, Server-Sent Events, or Firebase?
- [ ] **AI/ML** — Embeddings for semantic search (OpenAI, local, or vector DB)?
- [ ] **Deployment** — Vercel, Netlify, Railway, or self-hosted?
- [ ] **Monorepo** — Turborepo, Nx, or pnpm workspaces?

---

## Development Workflow

### Adding a New Admin Feature
1. Create column definitions in `admin-page/src/components/admin/columns/`
2. Add mock data in `admin-page/src/data/`
3. Create page component in `admin-page/src/pages/admin/`
4. Add route in `admin-page/src/App.jsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aritnatic/MyCampus](https://github.com/Aritnatic/MyCampus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
