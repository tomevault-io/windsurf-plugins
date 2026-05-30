---
trigger: always_on
description: **CleubAutomation (Lux Home Planner)** - Premium home automation planning & cost estimation platform. Users design smart homes room-by-room with real-time cost calculations and PDF proposals. Features role-based access, hybrid auth (Firebase + Supabase), and responsive UI.
---

````instructions
# AI Coding Agent Instructions - CleubAutomation

## Introduction

**CleubAutomation (Lux Home Planner)** - Premium home automation planning & cost estimation platform. Users design smart homes room-by-room with real-time cost calculations and PDF proposals. Features role-based access, hybrid auth (Firebase + Supabase), and responsive UI.

**Your Mission:** Fix bugs systematically, implement features sustainably, maintain production-quality standards.

---

## Core Principles

### ✅ DO's

**1. Broad-to-Narrow Problem Solving**
- Understand the full structure before implementing
- Identify root causes, not just symptoms
- **Always check related files** - types, services, components may need updates
- One change ripples: if you modify a type, update all components using it

**2. Database Operations (ALWAYS refer to SUPABASE_NOTES.md)**
- NEVER use local Supabase CLI
- Execute SQL directly in Supabase Dashboard > SQL Editor
- Update `src/supabase/types.ts` after schema changes
- Test in browser immediately after DB changes

**3. Quality Over Shortcuts**
- Follow existing patterns (no `any` types)
- Clean, focused code > quick patches
- Test changes before completing
- Verify no console errors after implementation

**4. Holistic Thinking**
- If you change a type → update consuming components
- If you modify a service → update calling code
- If you add a route → update navigation links
- Files are interconnected—never work in isolation

**5. Concise Communication**
- Keep responses short (1-2 lines max for summaries)
- Focus on WHAT changed & WHY
- **STRICTLY DO NOT create ANY MD files** (summary, analysis, guides, documentation, etc.) **WITHOUT explicit user request**
- Update SUPABASE_NOTES.md only for DB schema changes (must be explicitly requested)

### ❌ DON'Ts

- ❌ Never modify Firebase files without explicit request
- ❌ Don't copy-paste components—extract shared logic
- ❌ Never use local Supabase CLI migrations
- ❌ Don't ignore RLS policies—all inserts/updates need user_id validation
- ❌ Don't leave broken code—fix all errors before completing
- ❌ Never create one-off components when a universal one exists
- ❌ **NEVER create MD files, summaries, analysis docs, or guides without explicit user request** - Work silently, no documentation unless asked

---

## Critical Architecture

### Hybrid Auth System (Firebase + Supabase)
- **Firebase Auth:** Email/Google OAuth authentication
- **Supabase DB:** User profiles, projects, inquiries, inventory
- **AuthContext** (`src/contexts/AuthContext.tsx`): Syncs both systems
- **Pattern:** Auth state change → fetch profile from Supabase → auto-create if missing

### Service Layer (3 Main Services)

**User Service** (`src/supabase/userService.ts`)
- Profile CRUD, project list fetching

**Project Service** (`src/supabase/projectService.ts`)
- Full project lifecycle: create, save rooms/requirements, update costs
- JSONB fields: `client_info`, `property_details`, `rooms`, `sections`
- **Pattern:** Every update validates user_id matches (RLS enforced)

**Admin Service** (`src/supabase/adminService.ts`)
- Inquiries, inventory, testimonials, settings
- Checks `is_admin: true` before mutations

### Data Structure

```typescript
Project {
  client_info: { name, email, phone, address },
  property_details: { type, size, budget },
  rooms: Array<{ id, name, type, appliances }>,
  sections: Array<{ id, name, items }>,
  status: 'draft' | 'in-progress' | 'completed'
}
```

### Page Flow
1. `/` → `/intake` → `/room-selection` → `/requirements` → `/final-review` → `/planner`
2. `/history` - User's projects
3. `/admin/*` - Admin dashboard (role-protected)

---

## Development Workflow

### Database Changes (Dashboard-First Approach)
1. Write SQL → Supabase Dashboard > SQL Editor
2. Create migration file: `supabase/migrations/###_description.sql`
3. Verify schema in Tables view
4. Update `src/supabase/types.ts` to match schema
5. Update services if needed
6. Test in browser

### Code Standards
- **Types:** snake_case (DB), camelCase (UI components)
- **Error handling:** `console.log('✅ action', 'message')` | `console.error('❌ Error:', error)`
- **Validation:** Use TypeScript strictly—no `any` unless absolutely necessary
- **Components:** UI primitives in `src/components/ui/`, features in `src/components/features/`

### Git Workflow
- Branch: `feature/name` or `fix/bug-name`
- Commit: `feat: desc` | `fix: desc` | `update: desc`
- Always test: `npm run dev`

---

## Key Patterns & File Reference

| Path | Purpose |
|------|---------|
| `src/supabase/config.ts` | Supabase client init |
| `src/supabase/types.ts` | DB schema types (auto-generated) |
| `src/supabase/{user,project,admin}Service.ts` | Database operations |
| `src/contexts/AuthContext.tsx` | Auth state management |
| `src/types/*.ts` | UI layer types |
| `SUPABASE_NOTES.md` | **Full DB documentation, RLS policies, operations** |
| `GIT_WORKFLOW.md` | Git standards |

---

## Common Tasks

### Adding Inventory Category
1. Insert in `adminService.ts` SQL
2. Update `src/constants/inventory.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Krishna9827/CleubAutomation](https://github.com/Krishna9827/CleubAutomation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
