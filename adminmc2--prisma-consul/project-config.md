---
trigger: always_on
description: Marketing website + B2B discovery tool + document portal for PRISMA Consul, a pharma/healthcare consulting company. Self-hosted on IONOS VPS.
---

# PRISMA Consul - Web Platform

## Project Overview

Marketing website + B2B discovery tool + document portal for PRISMA Consul, a pharma/healthcare consulting company. Self-hosted on IONOS VPS.

**Live URL:** https://prismaconsul.com
**Hosting:** IONOS VPS (Ubuntu 24.04, nginx + Express.js + PM2)
**Database:** Neon PostgreSQL (serverless)
**File Storage:** Google Drive (via Workspace domain-wide delegation)

## Architecture

This is a monorepo with 3 frontend apps sharing one Express.js backend:

```
/                        → Marketing landing page (prismaconsul.com)
/apex                    → APEX Discovery Form (prismaconsul.com/apex) — requiere login
/hub                     → PRISMA Hub (prismaconsul.com/hub) — requiere login
/api/*                   → Express.js API backend
```

## Directory Structure

```
├── index.html                  # Landing page
├── aviso-legal.html            # Legal pages
├── cookies.html
├── privacidad.html
├── css/styles.css              # Landing page styles
├── js/main.js                  # Landing page scripts
├── images/                     # All media assets
│   ├── logos/                  # SVG logos, favicon
│   ├── team/                   # Team member photos
│   └── videos/                 # Marketing videos
├── apex/                       # APEX Discovery Form (self-contained SPA)
│   ├── index.html
│   ├── form.js                 # Main form logic (~3500 lines)
│   ├── form.css
│   ├── signal-detector.js
│   └── fonts/                  # Phosphor Icons (local)
├── portal/                     # PRISMA Hub (single-file SPA)
│   └── index.html              # Login + document management + admin panel
├── server/                     # Express.js backend
│   ├── server.js               # App setup, middleware, route mounting
│   ├── package.json            # All backend dependencies
│   ├── schema.sql              # PostgreSQL schema reference
│   ├── middleware/
│   │   ├── cors.js             # CORS headers (all routes)
│   │   └── auth.js             # JWT verification + requireAdmin middleware
│   ├── routes/
│   │   ├── portal.js           # Auth, upload, file management, user management, activity log
│   │   ├── apex.js             # Research, questions, form submission
│   │   └── ai.js               # Groq LLM chat + Whisper transcription
│   └── lib/
│       ├── pain-knowledge-base.js  # Pain/situation database (469 pains)
│       ├── google-drive.js     # Google Drive client + per-user folder helpers
│       └── fetch-timeout.js    # Fetch wrapper with AbortController
├── .env                        # Secrets (NOT committed)
├── .gitignore
└── .github/
    └── dependabot.yml          # Weekly dependency monitoring
```

## Tech Stack

- **Frontend:** Vanilla HTML/CSS/JS (no frameworks)
- **Fonts:** Quicksand (headings) + Source Sans 3 (body) via Google Fonts
- **Icons:** Phosphor Icons (local font files in `apex/fonts/`)
- **Backend:** Express.js with modular routes (server/routes/)
- **Database:** Neon PostgreSQL (`apex_submissions`, `portal_users`, `portal_files`, `portal_activity_log`)
- **Auth:** JWT (jsonwebtoken) + bcryptjs, 24h token expiry. Role-based: admin/user. Shared auth for APEX y Hub.
- **APIs:** Groq (LLM + Whisper), Tavily (web search), Claude API (questions)
- **File Storage:** Google Drive API via Service Account with domain-wide delegation
- **Email:** Gmail SMTP via Nodemailer

## Design System

- **Theme:** Dark (`--prisma-navy: #101B2C`, `--bg-primary: #1a2535`)
- **Accent:** `--tech-cyan: #31BEEF`
- **Secondary:** `--visionary-violet: #994E95`, `--soft-blue: #A1B8F2`
- **Text:** `--clinical-white: #FAF9F6`
- **Screen transitions:** opacity + translateY 400ms ease (APEX pattern)
- **Button styles:** `border-radius: 4px 25px 25px 4px` (primary), `25px 4px 4px 25px` (logout)

## Environment Variables

Required in `.env` on the VPS (`~/web-de-prisma/.env`):

```
# APIs
GROQ_API_KEY=
TAVILY_API_KEY=
CLAUDE_API_KEY=

# Database
DATABASE_URL=              # Neon PostgreSQL connection string

# Email
SMTP_USER=                 # Gmail address
SMTP_PASS=                 # Gmail app password

# Portal
PORTAL_SECRET=             # JWT signing secret
GOOGLE_SERVICE_ACCOUNT_KEY= # Full JSON of Google SA credentials
GOOGLE_DRIVE_FOLDER_ID=    # Target Drive folder ID
```

## Key Patterns

### APEX Form Flow
Login → Welcome → Company Input → Research (Tavily+Groq) → Swipe Cards → MaxDiff → Top4 → Phase 1 Questions → Phase 2 Adaptive → Pains → Audio → Contact → Thank You

### Business Type System
- `SITUACIONES_DISTRIBUIDOR` (16 cards, IDs: A-P) for distributors/pharma
- `SITUACIONES_CLINICA` (16 cards, IDs: CA-CP) for clinics
- Type detected from `research-company.js` field `detectado.es_clinica`

### PRISMA Hub (Portal)
- Login → Role detection → Admin gets 3 tabs (Documentos, Usuarios, Actividad), User gets 1 tab (Documentos)
- Upload: Drag files → Stage with type classification → Click "Subir" → Upload to user's Google Drive subfolder
- Admin can "view as user" to see any client's files
- Admin can create new users from the Usuarios tab

### Google Drive Integration
- Service Account with domain-wide delegation impersonates `info@prismaconsul.com`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adminmc2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
