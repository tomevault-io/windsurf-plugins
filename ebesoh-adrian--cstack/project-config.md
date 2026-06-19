---
trigger: always_on
description: >
---


# CodersHub Innovations — Company-as-a-Repo SKILL

> **One clone. One command. Your entire company runs.**

---

## 0. Quick-Start (for any contributor)

```bash
git clone https://github.com/codershublabs/company-os.git
cd company-os
cp .env.example .env.local        # fill in secrets (see §8)
pnpm install
pnpm db:migrate                   # applies all Supabase migrations
pnpm dev                          # boots all apps in parallel via Turbo
```

Open `http://localhost:3000` → main site  
Open `http://localhost:3001` → member portal  
Open `http://localhost:3002` → admin / ops dashboard  

---

## 1. Company DNA

| Field | Value |
|---|---|
| **Legal name** | CodersHub Innovations |
| **Brand name** | CodersHub Labs |
| **HQ** | Yaoundé, Cameroon 🇨🇲 |
| **Mission** | Empower African builders through hands-on engineering, structured incubation, and community collaboration |
| **Target market** | Africa-first → Global |
| **Languages** | Bilingual: English + French (i18n via `next-intl`) |
| **Stack** | TypeScript · Next.js 14 App Router · Supabase · Vercel · Turborepo · pnpm |
| **Payments** | MTN MoMo · Orange Money · Stripe · PayPal |
| **Integrations** | GitHub · Slack/Discord · WhatsApp (via Twilio/WhatsApp Cloud API) |

---

## 2. Ecosystem Arms

Each arm is a **Turborepo app or package**. They share a single Supabase instance but have isolated schemas/modules.

| Arm | Slug | Purpose | Port (dev) |
|---|---|---|---|
| Main Website | `web` | Marketing, landing pages, blog, SEO | 3000 |
| Member Portal | `portal` | Student/member dashboard | 3001 |
| Admin / OpsDesk | `admin` | Internal ops, tasks, docs, automation | 3002 |
| DevCore | `devcore` | Bootcamp & course management | 3003 |
| CodersHub Labs | `labs` | Startup incubation dashboard | 3004 |
| HubConnect | `hubconnect` | Community, networking, events | 3005 |
| DesignHub | `designhub` | Design services, portfolio, briefs | 3006 |
| TechSpire | `techspire` | Research arm, publications, reports | 3007 |
| OpsDesk | `opsdesk` | Internal task tracker, SOPs, HR | 3002 (shared with admin) |

---

## 3. Monorepo Folder Structure

```
company-os/
├── apps/
│   ├── web/              # Main marketing site (Next.js)
│   ├── portal/           # Member portal (Next.js)
│   ├── admin/            # Admin + OpsDesk (Next.js)
│   ├── devcore/          # Bootcamp management (Next.js)
│   ├── labs/             # Incubation dashboard (Next.js)
│   ├── hubconnect/       # Community platform (Next.js)
│   ├── designhub/        # Design arm (Next.js)
│   ├── techspire/        # Research arm (Next.js)
│   └── api/              # Shared API layer / edge functions (Hono.js)
│
├── packages/
│   ├── ui/               # Shared component library (Shadcn/ui base)
│   ├── db/               # Supabase client, types, migrations
│   ├── auth/             # Auth helpers (Supabase Auth)
│   ├── payments/         # MoMo + Stripe + PayPal abstraction layer
│   ├── i18n/             # English + French translations
│   ├── notifications/    # Email (Resend) + WhatsApp + Slack/Discord
│   ├── config/           # Shared ESLint, TS, Tailwind configs
│   └── utils/            # Shared utilities
│
├── supabase/
│   ├── migrations/       # All DB migrations (versioned)
│   ├── seed.sql          # Dev seed data
│   └── functions/        # Supabase Edge Functions
│
├── .github/
│   ├── workflows/        # CI/CD pipelines
│   └── CODEOWNERS        # Team ownership map
│
├── docs/
│   ├── architecture.md
│   ├── runbooks/         # Operational runbooks per arm
│   └── onboarding.md     # New team member guide
│
├── scripts/
│   ├── setup.sh          # First-time setup script
│   ├── seed-dev.ts       # Populate dev environment
│   └── deploy.sh         # Manual deploy trigger
│
├── turbo.json
├── pnpm-workspace.yaml
├── .env.example
└── package.json
```

---

## 4. Database Schema (Supabase / PostgreSQL)

All tables live in Supabase. Row Level Security (RLS) is **always on**.

### Core Tables

```sql
-- Identity
profiles           (id, user_id, full_name, avatar_url, role, arm, lang, created_at)
roles              (id, name, permissions[])   -- e.g. admin, mentor, student, founder

-- DevCore (Bootcamps)
cohorts            (id, name, arm, start_date, end_date, status, capacity)
enrollments        (id, user_id, cohort_id, status, payment_id, created_at)
modules            (id, cohort_id, title, order, content_url)
assignments        (id, module_id, title, due_date, max_score)
submissions        (id, assignment_id, user_id, content_url, score, feedback)

-- CodersHub Labs (Incubation)
startups           (id, founder_id, name, stage, sector, deck_url, status)
incubation_apps    (id, user_id, startup_id, status, reviewer_id, notes)
milestones         (id, startup_id, title, due_date, completed_at)

-- HubConnect (Community)
posts              (id, author_id, content, type, arm, likes, created_at)
events             (id, title, arm, date, location, rsvp_count, stream_url)
rsvps              (id, event_id, user_id, status)
mentorship_requests(id, mentee_id, mentor_id, status, topic, scheduled_at)

-- OpsDesk (Internal)
tasks              (id, assignee_id, title, status, priority, due_date, arm)
documents          (id, author_id, title, content, type, arm, version)
team_members       (id, user_id, arm, title, joined_at, status)

-- Payments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ebesoh-Adrian/cstack](https://github.com/Ebesoh-Adrian/cstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
