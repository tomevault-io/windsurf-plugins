---
trigger: always_on
description: > **Purpose:** Standard development rules and best practices
---

# GoWater Monorepo - Development Guidelines

> **Purpose:** Standard development rules and best practices
> **Last Updated:** 2026-03-18

> **Session Notes:** Water theme, P3 loading screen, page transitions, auth context, glass cards, TaskTimelineView compact layout

---

## Project Structure

```
gowater-monorepo/
├── apps/
│   ├── web/                 # Next.js 15 web application
│   └── mobile/              # Expo React Native application
├── packages/
│   └── types/               # Shared TypeScript types (@gowater/types)
├── infra/                   # Docker, Caddy, deployment configs (NOT YET CREATED — target architecture)
├── docs/                    # Documentation and references
│   ├── CODE_REFERENCE.md
│   ├── DATABASE_FIELDS_REFERENCE.md
│   ├── REUSABLE_REFERENCE.md
│   ├── TRANSITIONS_REFERENCE.md   # GSAP animation patterns catalog
│   ├── new-features.md            # P3 loading screen animation spec
│   └── persona-glass-cards.jsx    # Glass card style reference
└── CLAUDE.md                # This file
```

---

## Migration Status (Current → Target)

| Component            | Current (Production)               | Target (Hetzner VPS)                           | Status      |
| -------------------- | ---------------------------------- | ---------------------------------------------- | ----------- |
| Database             | Supabase (hosted PostgreSQL)       | Self-hosted PostgreSQL 16 in Docker            | Not started |
| Photo Storage        | Cloudinary                         | Hetzner Object Storage (S3-compatible)         | Not started |
| Watermarking         | Satori + Sharp → Cloudinary upload | Satori + Sharp → Hetzner Object Storage upload | Not started |
| Hosting              | Vercel (Next.js)                   | Hetzner VPS + Docker + Caddy                   | Not started |
| Infrastructure files | None (`infra/` dir empty)          | Dockerfile, docker-compose, Caddyfile, scripts | Not started |

**Why migrate from Cloudinary:** Cloudinary's transformation limits prevent custom watermark UI on photos (text overlay limitations). Satori+Sharp gives full JSX control over watermark design, and Hetzner Object Storage is cheaper for direct upload of pre-watermarked images.

**Important:** All `infra/` section content in this document describes the TARGET architecture. Do not assume these files or configs exist until this table is updated.

---

## Critical Rules

### 0. Use Opus 4.6

- **Always use Claude Opus 4.6** as the model for all AI-assisted development in this project
- Do not downgrade to Sonnet or Haiku for code generation tasks

### 1. No AI Attribution

- **Never mention AI, Claude, or any AI assistant** in commit messages, code comments, documentation, or anywhere in the codebase
- **No Co-Authored-By lines** referencing AI in commits
- Keep all contributions anonymous as standard developer work

### 2. Always Read Documentation First

Before making ANY changes, read the relevant docs:

- `docs/CODE_REFERENCE.md` - API endpoints, services, types
- `docs/DATABASE_FIELDS_REFERENCE.md` - Database schema, field names
- `docs/REUSABLE_REFERENCE.md` - Existing components, hooks, patterns

### 3. Never Hallucinate

- **Database fields:** Always use exact field names from `DATABASE_FIELDS_REFERENCE.md`
- **API endpoints:** Always verify endpoints exist in `CODE_REFERENCE.md`
- **Types:** Always import from `@gowater/types` or check existing type definitions
- **Components:** Check if component exists before creating new ones

### 4. Naming Conventions

| Context                 | Convention | Example                                      |
| ----------------------- | ---------- | -------------------------------------------- |
| Database fields         | snake_case | `user_id`, `check_in_time`, `break_duration` |
| TypeScript/JS variables | camelCase  | `userId`, `checkInTime`, `breakDuration`     |
| React components        | PascalCase | `TaskCard`, `AttendanceModal`                |
| CSS classes (web)       | kebab-case | `task-card`, `modal-header`                  |
| API routes              | kebab-case | `/api/attendance/edit-requests`              |
| File names (components) | PascalCase | `TaskCard.tsx`                               |
| File names (utilities)  | camelCase  | `formatDate.ts`                              |

### 5. Plan Before Executing

- State the full implementation plan before writing any code or modifying any file
- Get explicit approval before proceeding
- If scope changes mid-implementation, stop and re-plan

### 6. Production-Grade Clean Code

- No hacks, no shortcuts, no commented-out code, no dead code
- No bloating — do not add dependencies, abstractions, or utilities unless directly required
- No over-engineering — solve only what is asked, nothing more
- **Simple tasks require simple solutions** — if a feature is adding items to an existing sidebar list, don't create new pages/routes/layouts. Follow the existing pattern. If Leads/Events/Supplier are buttons that switch content inline, new categories must do the same — not navigate to separate pages
- Before creating new files, ask: can this be done by adding to an existing file? If yes, do that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tekkenfreya/gowater-hr-portal-monorepo](https://github.com/tekkenfreya/gowater-hr-portal-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
