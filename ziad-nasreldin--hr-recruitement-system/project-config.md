---
trigger: always_on
description: Do not make any changes until you have 95% confidence in what you need to build. Ask me follow-up questions until you reach that confidence.
---

@AGENTS.md

Do not make any changes until you have 95% confidence in what you need to build. Ask me follow-up questions until you reach that confidence.

---

## Purpose of This File

CLAUDE.md is the source of truth for this project. It stores stable decisions, architecture rules, and progress summaries — not conversations. Every architectural call recorded here is a paragraph Ziad never has to type again.

---

## Context Rules

- Do not use subagents for any exploration or research. If a task needs 3+ files or multi-file analysis, summarize the issue as a concise insight and present it directly.
- Save decisions, not conversations. If an architectural choice is made, record it here.
- Ask clarifying questions until 95% confidence is reached before writing any code or creating any files.

---

## Applied Learning

When something fails repeatedly, when Ziad has to re-explain, or when a workaround is found for a platform/tool limitation, add a one-line bullet here. Keep each bullet under 15 words. No explanations. Only add things that will save time in future sessions.

* Templates + Puppeteer for visual consistency. AI image gen for one-offs only.
* Agents fail silently on wrong paths. Always verify hardcoded paths.
* New skills need a validation step before rendering. First runs have data gaps.
* Google Slides 'autofit' crashes batchUpdate. Set font sizes explicitly.
* Windows Developer Mode required for symlinks (paperclip, etc.)
* Button (base-ui) has no asChild prop — use `buttonVariants` class on `<Link>` instead.
* `buttonVariants` must live in a separate file (no `"use client"`) — server pages can't import from client modules.
* Select component is a native `<select>` wrapper — no SelectContent/SelectItem/SelectTrigger.
* No Table UI component — use native HTML `<table>` elements directly.
* DropdownMenuTrigger has no asChild — style it directly with className.

---

## System Blueprint

Full-stack HR Recruitment CRM for managing candidates, job offers, follow-ups, commissions, and AI-assisted workflows. Built as a Next.js 16 App Router webapp with PostgreSQL + Prisma, NextAuth v5, Tailwind CSS 4, and Shadcn/UI.

**Target users:** Ziad (Super Admin) + Recruiters (subscription-gated access)

---

## Architecture Decisions

- **API layer:** Next.js Route Handlers under `src/app/api/` — one folder per resource
- **Auth:** NextAuth v5 Credentials provider, JWT sessions (30 days), middleware guards all dashboard routes
- **DB:** PostgreSQL via Prisma ORM — schema lives in `prisma/schema.prisma`, client output at `src/generated/prisma`
- **AI:** OpenAI API — Whisper for voice transcription, GPT-4o for English proficiency scoring + post generation
- **WhatsApp:** Twilio WhatsApp API (or fallback: WhatsApp Business Cloud API directly) — decision to be confirmed before Phase 8
- **File storage:** Voice notes stored via URL (external storage TBD — Cloudinary or S3-compatible); file upload handled server-side in API routes
- **Role system:** SUPER_ADMIN (Ziad) and RECRUITER. Middleware + `requireRole()` utility enforce access.

---

## Build Phases

### ✅ Phase 1 — Auth & Layout (COMPLETE)
- NextAuth Credentials login with bcryptjs password hashing
- Prisma schema with all models (User, Offer, Lead, VoiceNote, FollowUpReminder, Commission)
- Dashboard shell: Sidebar, Header, MobileSidebar, role-based nav
- Middleware: protects all routes, checks isActive + accountExpiresAt + trialEndsAt
- Placeholder pages for all sections

---

### ✅ Phase 2 — Offer Management (COMPLETE)
**Goal:** Recruiters can view available job offers. Admin can create, edit, and archive offers.

Pages:
- `/offers` — paginated offer list with filters (status, language, work model)
- `/offers/new` — create offer form (all schema fields)
- `/offers/[id]` — offer detail view
- `/offers/[id]/edit` — edit offer

API routes:
- `GET /api/offers` — list with query filters
- `POST /api/offers` — create (SUPER_ADMIN only)
- `GET /api/offers/[id]` — single offer
- `PUT /api/offers/[id]` — update (SUPER_ADMIN only)
- `PATCH /api/offers/[id]/status` — toggle ACTIVE / ON_HOLD

UI components: OfferCard, OfferForm, OfferStatusBadge, OfferFilters

---

### ✅ Phase 3 — Lead CRM & Pipeline (COMPLETE)
**Goal:** Recruiters can add candidates, track their progress through the 11-stage pipeline, and link them to offers.

Pages:
- `/leads` — lead list with filters (status, offer, recruiter, date range) + search
- `/leads/new` — create lead form
- `/leads/[id]` — lead detail: profile, status progression, voice notes, reminders, notes
- `/leads/[id]/edit` — edit lead info

API routes:
- `GET /api/leads` — list (recruiter sees own; admin sees all)
- `POST /api/leads` — create
- `GET /api/leads/[id]` — single lead
- `PUT /api/leads/[id]` — update fields
- `PATCH /api/leads/[id]/status` — advance/change pipeline stage (triggers reminder creation side-effect)
- `POST /api/leads/[id]/voice-notes` — upload voice note record (URL + metadata)
- `GET /api/leads/[id]/voice-notes` — list voice notes for lead

UI components: LeadCard, LeadForm, PipelineStageSelector, StatusBadge, LeadFilters, VoiceNoteUploader, LeadTimeline

Business logic:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ziad-NasrEldin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
