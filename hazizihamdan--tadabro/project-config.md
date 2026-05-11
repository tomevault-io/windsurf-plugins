---
trigger: always_on
description: A web application for managing Quran recitation (Tadarus) groups. Members are assigned daily page ranges, mark tasks complete with proof images, and track collective progress.
---

# Tada-Bro — Project Reference

A web application for managing Quran recitation (Tadarus) groups. Members are assigned daily page ranges, mark tasks complete with proof images, and track collective progress.

---

## Tech Stack

| Layer                  | Technology                                   |
| ---------------------- | -------------------------------------------- |
| Framework              | Next.js 16 (App Router)                      |
| Language               | TypeScript 5                                 |
| Styling                | Tailwind CSS v4 + shadcn/ui (new-york style) |
| Database               | PostgreSQL via Drizzle ORM                   |
| Auth                   | JWT (jose) stored in httpOnly cookie         |
| File Storage           | Cloudflare R2 (S3-compatible, AWS SDK v3)    |
| Password hashing       | bcryptjs                                     |
| Runtime env validation | @t3-oss/env-nextjs + Zod                     |
| Icons                  | @phosphor-icons/react                        |

---

## Project Structure

```
app/
  (app)/                    # Authenticated routes (layout with sidebar)
    dashboard/              # Today's tasks + overdue items
    group/
      page.tsx              # Group list
      new/                  # Create group (super_admin only)
      [groupId]/
        page.tsx            # Group detail + daily tasks
        edit/               # Edit group name, description, settings
        progress/           # Group progress table (all members)
        task/[taskId]/      # Complete a task (upload proof image)
    profile/                # View/edit profile, change password
    admin/                  # Admin stats panel (super_admin only)
    admin/users/            # All users list + admin actions (super_admin only)
  api/
    auth/login, logout, register, me
    dashboard/              # Dashboard data (groups + today tasks + overdue tasks)
    admin/stats/            # Admin stats panel data (super_admin only)
    groups/[groupId]/       # CRUD + members + tasks + progress
    groups/[groupId]/detail # Rich group detail (group + members + today tasks)
    tasks/[taskId]/complete
    users/, users/[userId]
    profile/, profile/password
    avatar/, upload/
    invite/[token]/
  globals.css               # Tailwind + shadcn CSS variables
  layout.tsx
lib/
  db/
    index.ts                # Drizzle + pg Pool singleton
    schema.ts               # All table definitions + relations
    migrations/             # Drizzle SQL migrations
  utils/
    date.ts                 # todayLocal, formatDate, isToday, isPast, addDays, daysBetweenInclusive
    cn.ts                   # clsx + tailwind-merge helper
    invite.ts               # Invite token generation/hashing
components/
  auth/                     # LoginForm, RegisterForm
  group/                    # GroupCard, ProgressTable, TaskCard, etc.
  layout/                   # Sidebar, BottomNav
  ui/                       # Shared UI: Input, Button, Card, Badge, Textarea, Table, DropdownMenu, ...
scripts/
  seed-super-admin.ts       # Seeds initial super_admin user
  check-r2.ts               # Verifies R2 credentials + bucket reachability
  regenerate-group-tasks.ts # Regenerates daily tasks for a group
types/
  session.ts                # SessionPayload, SessionUser interfaces
env.ts                      # Validated env vars via @t3-oss/env-nextjs
```

---

## Database Schema

### Enums

- `user_role`: `super_admin | admin | member`
- `objective_type`: `whole_quran | page_range | surah_range | juz_range`
- `task_status`: `pending | complete | missed`

### Tables

**users**
| Column | Type | Notes |
|---|---|---|
| id | serial PK | |
| username | varchar(50) unique | |
| full_name | varchar(150) | |
| email | text unique | nullable (legacy accounts may not have it) |
| ic_number_encrypted | text unique | nullable; AES-256-GCM encrypted; format `hex(iv):hex(authTag):hex(ciphertext)` |
| ic_number_hash | varchar(64) unique | nullable; HMAC-SHA256 for uniqueness checks without decryption |
| password_hash | text | bcrypt |
| role | user_role | default `member` |
| avatar_url | text | nullable; R2 presigned URL key |
| created_at / updated_at | timestamptz | |

**groups**
| Column | Type | Notes |
|---|---|---|
| id | serial PK | |
| name | varchar(100) | |
| description | text | nullable |
| admin_id | int FK → users | |
| member_count | int | enforced ≤ 10 in API |
| objective_type | objective_type enum | |
| objective_meta | jsonb | null for whole_quran; `{startPage,endPage}` for page_range; `{startSurah,endSurah,startPage,endPage}` for surah_range; `{startJuz,endJuz,startPage,endPage}` for juz_range |
| total_pages | int | pre-computed at creation |
| start_date / end_date | date | YYYY-MM-DD |
| is_active | boolean | default true |
| allow_member_view | boolean | default false; allows members to see each other's proof images |
| created_at / updated_at | timestamptz | |

**group_members**

- `(group_id, user_id)` unique index
- `member_index` (0-based) determines page assignment order

**daily_tasks**

- `(group_id, user_id, task_date)` unique index

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaziziHamdan/tadabro](https://github.com/HaziziHamdan/tadabro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
