---
trigger: always_on
description: - **Backend:** Ruby on Rails 8.0 (Modern Monolith, Propshaft)
---

# Project Guide: IndustrialProfi — B2B SaaS Platform

## Tech Stack

### Active
- **Backend:** Ruby on Rails 8.0 (Modern Monolith, Propshaft)
- **Frontend:** React 18 + TypeScript 5.7 (strict) via Inertia.js 2.0
- **Build Tool:** Vite 5 + vite-plugin-ruby
- **Database:** PostgreSQL 16
- **Styling:** Tailwind CSS 3.4 + shadcn/ui (Radix primitives) + Headless UI
- **Graphs:** @xyflow/react 12.3 (React Flow)
- **Icons:** Lucide React
- **i18n:** react-i18next (ru + en)
- **Dates:** date-fns 4

### Planned
- **Components:** Magic UI (Marketing), Aceternity UI (Hero effects)
- **Typography:** Geist Sans (Vercel)

### Backend Stack
- **Auth:** Custom session-based (bcrypt) + Pundit (authorization policies)
- **Multi-tenancy:** ActsAsTenant (organization isolation)
- **Audit:** PaperTrail (compliance versioning)
- **Jobs:** Solid Queue (no Redis)
- **Cache:** Solid Cache
- **WebSockets:** Solid Cable
- **Email:** ActionMailer + letter_opener (dev)
- **Pagination:** Pagy
- **Deployment:** Kamal 2 + Docker + Thruster

### Runtime Versions
- **Ruby:** 3.4.1
- **Node:** not pinned (Vite 5, works with Node 18+)

## Domain Model

```
Organization (tenant root)
├── has_many Users (employee | manager | owner)
├── has_many JobTitles
├── has_many RoadmapAssignments
│
User
├── belongs_to Organization (acts_as_tenant)
├── has_many RoadmapAssignments → assigned_roadmaps (through)
├── has_many UserProgresses (skill completion tracking)
├── has_many RoadmapFavorites → favorite_roadmaps (through)
├── has_many authored_roadmaps (Roadmap.author_id)
├── has_many verified_progresses (UserProgress.verified_by_id)
│
Roadmap (organization_id nullable — nil = global public template)
├── belongs_to Organization (optional, acts_as_tenant has_global_records: true)
├── belongs_to author: User (optional)
├── belongs_to forked_from: Roadmap (self-referential fork chain)
├── has_many Skills (counter_cache: skills_count)
├── has_many RoadmapAssignments → assigned_users (through)
├── has_many RoadmapFavorites (counter_cache: favorites_count)
│
Skill
├── belongs_to Roadmap
├── belongs_to PermitTemplate (optional — auto-sets skill_type to "permit")
├── has_many SkillDependencies (from/to — directed graph edges)
├── types: skill | permit | milestone
├── stage (integer) — visual grouping for React Flow layout
│
UserProgress (core training record: user × skill)
├── belongs_to User
├── belongs_to Skill
├── belongs_to verified_by: User (optional — manager who approved)
├── status flow: todo → in_progress → pending_review → completed
├── permit fields: certificate_number, issued_at, expires_at, issuing_authority
│
PermitTemplate (global, no org — shared permit definitions)
├── code (unique, e.g. "ELECTRICAL_1"), expiration_months, country_code
```

Key relationships: A manager assigns a User to a Roadmap (via RoadmapAssignment). The user progresses through each Skill in that Roadmap (via UserProgress). For permit-type skills, a manager verifies completion and the system tracks certificate expiry.

## Directory Structure
- `app/frontend/pages/` — Inertia page components, grouped by domain (Auth, Dashboard, Landing, Roadmaps, Org)
- `app/frontend/components/ui/` — shadcn/Radix base components
- `app/frontend/components/roadmap/` — XyFlow custom nodes and graph visualization
- `app/frontend/components/dashboard/` — Dashboard-specific widgets
- `app/frontend/components/landing/` — Landing page marketing sections
- `app/frontend/components/layout/` — Layout primitives (Sidebar)
- `app/frontend/components/shared/` — Reusable components (LanguageSwitcher, FlashMessages, TagsInput)
- `app/frontend/layouts/` — MainLayout (nav + sidebar), AuthLayout (centered card)
- `app/frontend/lib/` — Utilities: route helpers, i18n config, graph layout, cn()
- `app/frontend/types/` — TypeScript interfaces mirroring Rails serializer output
- `app/frontend/locales/` — Translation JSON files (en.json, ru.json)
- `app/frontend/entrypoints/` — Vite entry points (inertia.tsx, application.css)
- `app/controllers/` — Rails controllers with Inertia renders
- `app/policies/` — Pundit authorization policies
- `app/mailers/` — UserMailer, ManagerMailer
- `app/jobs/` — ActiveJob tasks (Solid Queue)

## Coding Standards & Rules
- **TypeScript Strict:** All frontend code in TypeScript strict mode. Define interfaces in `types/`.
- **i18n First:** All user-facing strings via `t()` from react-i18next. No hardcoded text.
- **Centralized Routes:** All URLs via `routes.*` helpers from `lib/routes.ts`. No hardcoded paths.
- **Performance First:**
  - Prioritize CSS animations (`tailwindcss-animate`) over JS-based ones.
  - Use `React.lazy` for components below the fold.
- **Inertia.js:** Always prefer SSR-friendly patterns. Use `router.*` and `<Link>` for navigation.
- **Styling:** Tailwind classes only. Global variables in `application.css`. Dark mode first aesthetic.
- **Multi-tenancy:** Always scope queries through `current_organization`. Never bypass tenant isolation.
- **Authorization:** Every controller action must be authorized via Pundit (`verify_authorized`).
- **React Flow:** Use XyFlow v12+ patterns. `nodeTypes` declared outside components. Custom nodes wrapped in `memo()`.
- **No Hardcoded Secrets:** Use Rails credentials or environment variables.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreiyurik/v7-industrialprofi-platform-v7](https://github.com/andreiyurik/v7-industrialprofi-platform-v7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
