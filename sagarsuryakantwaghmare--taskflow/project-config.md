---
trigger: always_on
description: Guidance for working in this repository. Read this first to get the overall picture
---

# CLAUDE.md

Guidance for working in this repository. Read this first to get the overall picture
before changing code.

## What this project is

**TaskFlow** is a full-stack task management application.

| Part | Stack | Deployed at |
|------|-------|-------------|
| `client/` | React 18 + Vite + Tailwind CSS + shadcn/ui | Vercel |
| `server/` | Node.js + Express + MongoDB (Mongoose) | Render |
| `automation/` | Java 21 + Maven + TestNG + REST Assured + Selenium | Runs locally / CI |

The frontend and backend are written and maintained by the repository owner. The
`automation/` module is a separate Maven project that tests the deployed
application from the outside — it never imports app code.

## Repository layout

```
taskflow/
├── client/         # React frontend (Vite dev server on :5173)
├── server/         # Express API (default :5000)
├── automation/     # Java test automation suite (API + UI)
├── screenshots/    # Images used by README.md
├── package.json    # Root scripts to run client and server together
├── README.md       # Public project documentation
└── CLAUDE.md       # This file
```

## Commands

The primary shell here is **Windows PowerShell 5.1**, which does not support the
`&&` chaining operator — each `cd` goes on its own line.

```bash
# Frontend + backend together (from the repo root)
npm run dev

# Frontend only
cd client
npm run dev

# Backend only
cd server
npm run dev

# Automation - everything
cd automation
mvn test

# Automation - API only (no browser opens)
mvn test -Papi

# Automation - UI only
mvn test -Pui

# Automation - single class
mvn test -Dtest=TaskTest

# Automation - by group: smoke, regression, api, ui, auth, task, category
mvn test -Dgroups=smoke
```

Suite selection uses Maven **profiles** (`-Papi` / `-Pui`), not `-Dsuite=...`.
PowerShell splits an unquoted `-D` argument containing a file extension, so
`mvn test -Dsuite=testng-api.xml` fails with *Unknown lifecycle phase ".xml"*.
If you do need the property form, quote it: `mvn test "-Dsuite=testng-api.xml"`.

## Backend API contract

All responses are JSON. Protected routes need `Authorization: Bearer <jwt>`.
JWTs are issued at register and login and are valid for 7 days.

### Auth — `/api/auth`

| Method | Path | Auth | Success | Notes |
|--------|------|------|---------|-------|
| POST | `/register` | no | `201` | Returns `data.user` + `data.token`. Duplicate email returns **`401`**, not 409. Missing fields return `400`. |
| POST | `/login` | no | `200` | Returns `data.user` + `data.token`. Bad credentials return `401` *Invalid credentials*. |
| GET | `/profile` | yes | `200` | Returns `data.user`. Password is stripped by the model's `toJSON`. |
| POST | `/logout` | yes | `200` | Stateless — the client just drops the token. |

### Tasks — `/api/todos`

> The UI calls these "tasks" but every route, model and message says **todo**.
> There is no `/api/tasks` endpoint.

| Method | Path | Success | Notes |
|--------|------|---------|-------|
| GET | `/` | `200` | Returns `data.todos` + `data.pagination`. Query params: `page`, `limit`, `completed`, `priority`, `category`, `search`. |
| POST | `/` | `201` | Returns `data.todo`. |
| GET | `/:id` | `200` | `404` *Todo not found* when missing. |
| PUT | `/:id` | `200` | Runs mongoose validators. |
| DELETE | `/:id` | `200` | *Todo deleted successfully*. |
| PATCH | `/:id/toggle` | `200` | Flips `completed`. |

Todo validation rules that tests rely on:

- `title` — required, max 100 characters
- `description` — **required, minimum 10 characters**, max 500
- `priority` — `low` | `medium` | `high`, defaults to `medium`
- `category` — free text, defaults to `general`
- `dueDate` — optional, must be today or in the future

Validation failures come back as `400` with the raw mongoose message.

### Categories — `/api/categories`

| Method | Path | Success | Notes |
|--------|------|---------|-------|
| POST | `/` | `201` | `400` if the name is missing or already used. |
| GET | `/` | `200` | Returns four hard-coded defaults (`Work`, `Personal`, `Shopping`, `Health`) with ids `default-0`…`default-3`, followed by the user's own categories. |
| PUT | `/:id` | `200` | `403` on a `default-*` id, `404` if not owned. |
| DELETE | `/:id` | `200` | `403` on a `default-*` id, `404` if not owned. |

`color` must be a valid hex colour (`#3b82f6`). The default categories exist only
in the controller — they are never stored in MongoDB.

### Auth failure messages

These exact strings are asserted by the automation suite, so do not reword them
without updating the tests:

- No token → `Access denied. No token Provided.`
- Bad token → `Invalid token`
- Bad login → `Invalid credentials`
- Duplicate register → `Account is already existed with this account`

## Frontend notes

- Routes live in `client/src/App.jsx`: `/`, `/login`, `/signup`, `/dashboard`,
  `/about`, `/privacy`, `/terms`, and a catch-all 404.
- `client/src/lib/api.js` is a bare axios instance using `VITE_API_URL`. There is
  no interceptor — each dashboard call attaches the `Authorization` header itself.
- The JWT is kept in `localStorage` under `token`. The dashboard redirects to
  `/login` when it is missing.
- After a successful login the form shows a banner and redirects to `/dashboard`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SagarSuryakantWaghmare/taskflow](https://github.com/SagarSuryakantWaghmare/taskflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
