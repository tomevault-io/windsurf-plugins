---
trigger: always_on
description: > This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.
---

# Agent Instructions for Autodirecto

> This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

> **📖 REQUIRED READING: [`PROJECT_OVERVIEW.md`](./PROJECT_OVERVIEW.md)** — This is your primary knowledge base. It explains the full architecture, every folder, every system (SimplyAPI, MrCar, Funnels, Camera PWA, ChileAutos), all API routes, database tables, and the complete business flow. **Read it before diving into any code work.**

## What is Autodirecto?

**Autodirecto** is a car consignment platform for Chile. We help people sell their cars by:
1. AI-powered valuation (MrCar engine)
2. Professional photography (Camera PWA with ghost overlays)
3. Digital contract signing
4. Multi-channel publishing (autodirecto.cl + ChileAutos.cl)
5. CRM pipeline management (leads → consignment → inspection → sale)

**Core Systems:**
- **Next.js Frontend** (autodirecto.vercel.app) — Public site, catalog, vehicle detail pages
- **SimplyAPI** (autodirectocrm.vercel.app) — Flask backend + Alpine.js CRM dashboard (THE BRAIN)
- **Camera PWA** (cameracar.vercel.app) — Mobile photo capture with AR-like overlays
- **MrCar** (mrcar-cotizacion.vercel.app) — AI pricing engine for Chilean market
- **ChileAutos Integration** — Publish to Chile's largest marketplace + receive buyer leads
- **Supabase** — PostgreSQL database + storage for all data and photos

You operate within a 3-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**
- Basically just SOPs written in Markdown, live in `directives/`
- Define the goals, inputs, tools/scripts to use, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**
- This is you. Your job: intelligent routing.
- Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings
- You're the glue between intent and execution. E.g you don't try scraping websites yourself—you read `directives/scrape_website.md` and come up with inputs/outputs and then run `execution/scrape_single_site.py`

**Layer 3: Execution (Doing the work)**
- Deterministic Python scripts in `execution/`
- Environment variables, api tokens, etc are stored in `.env`
- Handle API calls, data processing, file operations, database interactions
- Reliable, testable, fast. Use scripts instead of manual work. Commented well.

**Why this works:** if you do everything yourself, errors compound. 90% accuracy per step = 59% success over 5 steps. The solution is push complexity into deterministic code. That way you just focus on decision-making.

## Key Context for Autodirecto

**Database & Data Flow:**
- All data lives in **Supabase (PostgreSQL)** at `kqympdxeszdyppbhtzbm.supabase.co`
- `db.py` is a compatibility layer that translates SQL-like queries to Supabase REST API
- **Bidirectional sync**: Changes to owner info in CRM leads auto-sync to consignaciones and vice versa
- Matching logic: Supabase ID → plate → RUT → phone

**Critical Tables:**
- `consignaciones` — Core entity (consigned vehicles) with ChileAutos integration fields
- `listings` — Published catalog entries with `chileautos_id`, `chileautos_status`
- `crm_leads` — All leads with `ai_consignacion_price` (saved, never recalculated!)
- `compradores` — Buyers with `source` field (manual/chileautos)
- `appraisals` — Vehicle inspections
- `camera_jobs` — Photo session tokens for Camera PWA
- `crm_settings` — Key-value config (WhatsApp, ChileAutos creds)

**Business Logic Rules:**
1. **AI pricing is cached**: When "Tasar con IA" runs in Funnels, price saves to `crm_leads.ai_consignacion_price`. Wizard reuses this—NO recalculation!
2. **Auto-unpublish**: When consignación status → `vendida` or `cancelada`, automatically unpublishes from ChileAutos
3. **ChileAutos creds in DB**: Stored in `crm_settings` table (not env vars) for UI configurability
4. **Token-based camera auth**: No login—CRM generates unique token → inspector opens URL → photos upload to correct Supabase folder

**API Architecture:**
- Next.js API routes (`/api/*`) are **proxies only**—no business logic except `normalizeRow()` in listings
- SimplyAPI `app.py` (~5,500 lines) contains ALL business logic
- Camera PWA uploads directly to Supabase Storage (bypasses backend)

**Where Things Live:**
- `src/app/` — Next.js pages (landing, catalog, wizard, detail pages)
- `src/app/api/` — Next.js API proxies
- `src/app/components/` — React components
- `SimplyAPI/app.py` — All backend routes & logic
- `SimplyAPI/templates/index.html` — Entire CRM dashboard (~6,000 lines Alpine.js SPA)
- `SimplyAPI/Funnels/` — Facebook lead scraping system
- `SimplyAPI/directives/` — SOPs for AI agents
- `SimplyAPI/execution/` — Deterministic Python scripts
- `Mrcar/` — AI pricing engine (Flask app)
- `camera app/web-deploy/` — Camera PWA (Vite build)

## Operating Principles

**1. Check for tools first**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dryanez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
