---
trigger: always_on
description: > Make any coding agent productive in this repo immediately. Zero spaghetti. No regressions. Mobile-first. Elegant.
---

# AGENTS.md — ChravelApp update to the file

> Make any coding agent productive in this repo immediately. Zero spaghetti. No regressions. Mobile-first. Elegant.

-----

## 0. NON-NEGOTIABLES (read first, enforce always)

1. **No regressions.** Every behavior change requires an explanation + verification checklist. If tests exist, run them. If not, add one.
1. **Small diffs win.** Surgical fixes over refactors. If a refactor is required, phase it — behavior-preserving commits first.
1. **One source of truth.** No duplicate types, constants, or business rules. If you find duplication, consolidate it cleanly.
1. **Type safety > vibes.** No `any` unless it's an intentional boundary — comment it with `// intentional: <reason>`.
1. **Mobile-first UX.** Every UI change is evaluated for small screens + PWA constraints (tap targets ≥44px, scroll, performance).
1. **Performance is a feature.** No heavy re-renders, excessive providers, chatty queries, or unnecessary component mounts.
1. **Demo mode is sacred.** Mock data is NEVER modified. Authenticated mode uses parallel real data paths. This is a hard architectural invariant.
1. **Anti-Goldfish Protocol.** Before coding: read `claude-progress.txt` + recent git log. Implement ONE feature → E2E test → commit → update breadcrumbs. Never multi-task features.
1. **Learn from trajectories.** Before non-trivial tasks, read `DEBUG_PATTERNS.md` + `LESSONS.md`. After completing, update memory files with evidence-backed learnings. See CLAUDE.md § AGENT LEARNING PROTOCOL.

-----

## 1. WHAT CHRAVEL IS

Chravel is the **group travel coordination layer** — consolidating the core trip objects into one place so groups aren't juggling 15+ apps:

|Module               |Description                                 |
|---------------------|--------------------------------------------|
|**Chat / Broadcasts**|Group messaging + announcement channel      |
|**Calendar**         |Shared trip itinerary & scheduling          |
|**Places & Links**   |Curated location bookmarks (not OTA booking)|
|**AI Concierge**     |In-app conversational AI for trip planning  |
|**Polls**            |Group decision-making                       |
|**Tasks**            |Trip to-do management                       |
|**Payments**         |Expense tracking (not a payment processor)  |
|**Media**            |Shared photo/video hub                      |

**Key product surfaces:**

- **Consumer** (free/paid): friend groups, families, travel squads
- **Pro / Enterprise**: touring teams, sports orgs, corporate travel — requires roles, permissions, operational reliability
- **Events**: growth loop / viral engine (event link → guests → new users)

> ⚠️ Chravel is coordination, NOT a full OTA booking engine. Do not build booking aggregation workflows that drag into licensing/compliance traps unless explicitly instructed.

-----

## 2. STACK

```
Frontend:    React 18, TypeScript, TanStack Query, Zustand
Styling:     Tailwind CSS (mobile-first)
Backend:     Supabase (Postgres + RLS + Edge Functions)
Deployment:  Vercel
Mobile:      Capacitor (iOS wrapper) / PWA
Payments:    Stripe
Maps:        Google Maps
Auth:        Supabase Auth + Firebase (notifications)
Repo:        github.com/MeechYourGoals/chravel
```

-----

## 3. DOCS INDEX (navigate before touching)

```
src/
  components/      → Shared UI components
  pages/           → Route-level views (map to product modules above)
  hooks/           → Data fetching + state hooks (single source per feature)
  lib/             → Supabase client, utility functions
  types/           → ALL shared TypeScript types live here — do not duplicate
  store/           → Zustand slices

Key files:
  claude-progress.txt       → Anti-Goldfish breadcrumbs — READ FIRST
  supabase/migrations/      → Schema source of truth
  supabase/functions/        → Edge functions
  capacitor.config.ts       → iOS wrapper config
  vercel.json               → Deployment config
```

> If you don't know where something lives — search. `rg "symbolName"`. Do not guess file paths or APIs.

-----

## 4. GOLDEN WORKFLOW

### Step A — Reproduce & Observe

- Identify exact user path and expected behavior
- Capture: current behavior | desired behavior | definition of done (DoD)

### Step B — Locate the single source of truth

- Types → `src/types/`
- Validators → one place (find it, don't create a second)
- API calls → centralized in hooks or lib — not inline in components
- UI components → `src/components/`

If you find multiples of any of these, pick canonical and delete/redirect the duplicates.

### Step C — Implement the smallest safe change

- Prefer: pure functions, narrow component changes, small testable deltas
- Avoid: "while I'm here" refactors, style churn, mass renames

### Step D — Prove it didn't break

```bash
npm run typecheck && npm run lint && npm run build
```

Run relevant unit/integration tests if present. Add at least one test if none exist for the touched area. Document manual verification steps for key flows.

-----

## 5. CODE QUALITY GATES

### 5.1 No duplicate logic / dead code

Before finalizing any change in an area:

- `rg "symbolName"` to verify import count
- Consolidate duplicate helpers/hook variants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Chravel-Inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
