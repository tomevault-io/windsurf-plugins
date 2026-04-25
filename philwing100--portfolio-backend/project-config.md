---
trigger: always_on
description: A Node.js/Express REST API for a personal productivity app. Handles user auth, list management, habit streak tracking, and flashcard study. Deployed to Vercel (serverless), backed by Supabase (PostgreSQL).
---

# Portfolio Backend — Agent Context

A Node.js/Express REST API for a personal productivity app. Handles user auth, list management, habit streak tracking, and flashcard study. Deployed to Vercel (serverless), backed by Supabase (PostgreSQL).
The frontend can be found at [github.com/philwing100/](https://github.com/philwing100/portfolio-vue-frontend)
Which is useful for its api docs.

---

## Stack

- **Runtime:** Node.js, Express 4
- **Database:** PostgreSQL via `pg` (Supabase), all queries go through PostgreSQL functions
- **Auth:** Google OAuth 2.0 (passport) + dual JWT (access + refresh tokens)
- **Deployment:** Vercel (`vercel.json` routes everything to `app.js`)

---

## Project Layout

```
app.js                    Entry point, middleware setup, JWT guard
helpers.js                Date utility functions
databaseConnection/
  database.js             PostgreSQL connection pool (Supabase via DATABASE_URL)
routes/
  index.js                Aggregates all routes under /api
  auth.js                 Google OAuth + JWT token endpoints
  lists.js                List CRUD
  streaks.js              Habit streak tracking
  flashcards.js           Flashcard sets, cards, and SM-2 study sessions
supabase/
  migrations/             4 .sql files: tables, functions, flashcard tables, flashcard functions
stored_procedures/        Legacy MySQL .sql files — no longer used
```

---

## Environment Variables

```
# Database
DATABASE_URL              Supabase connection string (Transaction Pooler, port 6543)

# Auth
JWT_SECRET                  Access token signing secret
JWT_REFRESH_SECRET          Refresh token secret (falls back to JWT_SECRET)
googleClientId              Google OAuth client ID
googleClientSecret          Google OAuth client secret

# Server
FRONTENDPORT                Defaults to 3000; != 3000 means Vercel/prod
NODE_ENV                    "production" enables secure/sameSite=none cookies
```

---

## Auth Flow

**Login:**
1. `GET /api/auth/google` → redirects to Google
2. `GET /api/auth/google/callback` → generates access token (24h) + refresh token (7d)
3. Refresh token stored in HTTP-only cookie; access token passed via redirect URL param

**Token Refresh:**
- `POST /api/auth/refresh` — reads refresh token from cookie, `x-refresh-token` header, or body
- Rotates refresh token (old hash revoked, new hash saved to `sessions` table)

**Protected Routes:**
- All `/api/*` routes except `/api/auth/*` and `/api/*/logout` require `Authorization: Bearer <token>`
- Middleware decodes token and sets `req.user = { id, email }`

**Logout:**
- `POST /api/auth/logout` — revokes refresh token hash in DB, clears cookie

---

## API Endpoints

### Auth (`/api/auth`)
| Route | Method | Purpose |
|---|---|---|
| `/google` | GET | Start OAuth |
| `/google/callback` | GET | OAuth callback |
| `/refresh` | POST | Rotate refresh token |
| `/check-auth` | GET | Validate token, return user |
| `/logout` | POST | Revoke refresh token |

### Lists (`/api/lists`) — action-dispatch POST
| Action | Purpose |
|---|---|
| `createList` | Upsert list by `parent_page` + `date` |
| `getList` | Fetch list by `parent_page` + `date`, or by `title` (legacy) |

### Streaks (`/api/streaks`) — action-dispatch POST
| Action | Purpose |
|---|---|
| `getStreaks` | Fetch all streaks; auto-resets any streak not updated in 7+ days |
| `incrementStreak` | Increment count (idempotent — once per day) |
| `updateStreak` | Upsert streak metadata |
| `deleteStreak` | Delete a streak |

### Flashcards (`/api/flashcards`) — REST style
| Route | Method | Purpose |
|---|---|---|
| `/sets` | GET | Paginated list of sets (`?page=1&limit=20`) |
| `/sets` | POST | Create a set (`{ title, description?, tags? }`) |
| `/sets/bulk` | POST | Create set + cards from delimited text |
| `/sets/:setId` | GET | Set metadata + all cards |
| `/sets/:setId` | PUT | Update set metadata |
| `/sets/:setId` | DELETE | Delete set and all its cards (cascade) |
| `/sets/:setId/cards` | POST | Add one or more cards (`{ term, definition }` or array) |
| `/cards/:cardId` | PUT | Update card content; resets SM-2 if content changed |
| `/cards/:cardId` | DELETE | Delete a card |
| `/cards/:cardId/review` | POST | Submit SM-2 grade (0–5), persists new state |
| `/study` | GET | Study session: due cards + new cards (`?tags=&new_limit=20`) |

---

## Database Schema (key tables)

**`users`**
- `user_id` (SERIAL PK), `google_id` (VARCHAR UNIQUE), `email` (VARCHAR UNIQUE)

**`sessions`**
- `session_id` (VARCHAR PK), `user_id` (FK), `expires` (BIGINT), `data` (JSONB)
- `refresh_token_hash`, `refresh_token_expires`, `refresh_token_revoked`, `refresh_token_replaced_by`

**`lists`**
- `list_id` (SERIAL PK), `user_id` (FK), `parent_page` (VARCHAR), `list_date` (DATE)
- `lists_json` (JSONB), `list_timestamp` (TIMESTAMP), `last_modified` (auto-update trigger)
- UNIQUE: `(user_id, parent_page, list_date)`

**`listitems`**
- `item_id` (SERIAL PK), `parent_list_id` (FK), `parent_item_id` (nullable FK — nesting)
- `text_string`, `completed`, `scheduled_date`, `due_date`, `recurring_task`, etc.

**`streaks`**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/philwing100) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
