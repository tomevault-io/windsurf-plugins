---
trigger: always_on
description: Here is the complete updated CLAUDE.md file in a single copy-pasteable block:
---

Here is the complete updated CLAUDE.md file in a single copy-pasteable block:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Madina Collar is a fabric sales/ordering system with **four interfaces in this repo**, each in its own top-level folder:

1. **Mobile app** (`mobile/`) — Expo-managed React Native app. The original/primary client.
2. **Web app** (`web/`) — Next.js 14 (Pages Router) app that **deliberately re-implements the same ordering flow** (auth → catalog → product/size-selection → checkout → orders) as the mobile app, hitting the same backend API. Treat it as a parallel client, not a shared codebase — it duplicates logic rather than importing from `mobile/src/` (different framework, different storage APIs).
3. **Admin panel** (`mc-admin-panel/`) — React 19 + Vite + Tailwind v4 SPA for staff/admins (`role: 'admin' | 'superadmin'`): dashboard analytics, order management, fabric quality/style/attribute/price-matrix CRUD, customer list, staff management. Hits the same backend's `/api/admin/*` and `/api/auth/*` routes. Independently maintained from `mobile/`/`web/` — its own React/Vite tooling, its own re-implementation of admin-facing screens.
4. **Backend** (`backend/`) — Express + Sequelize (PostgreSQL) API shared by all three frontends.

Each of `mobile/`, `web/`, `mc-admin-panel/`, and `backend/` has its own `package.json`/`node_modules` and is developed independently — there is no shared root-level `package.json` or workspace tooling tying them together.

Because the mobile and web apps are independently-maintained copies of the same flow, **a business-logic or bug fix (e.g. price lookup, cart key format, order payload shape) usually needs to be applied twice** — once in `mobile/src/` (JS) and once in `web/src` (TS) — unless it's purely a backend fix. `mc-admin-panel/` is a separate flow (admin/staff-facing, not the customer ordering flow) and does not duplicate mobile/web logic, but it does share the backend's `/api/admin/*` contract — a backend admin-route change (path, payload shape, response shape) must be checked against `mc-admin-panel/src/pages/*.jsx` too.

## Production Deployment

The app is **live in production**:
- **Web** (customer app): https://almadina.site
- **Admin panel**: https://admin.almadina.site
- **Backend API**: https://api.almadina.site

Infrastructure:
- Backend runs on a **DigitalOcean droplet** at `168.144.74.52` (Bangalore region).
- **PostgreSQL** runs locally on the droplet (user: `madina`, database: `madina_collar`).
- **PM2** manages both Node processes: `madina-backend` (port 5000) and `madina-web` (port 3000).
- **Nginx** routes traffic by host: `almadina.site` → Next.js (web app, port 3000), `api.almadina.site` → Express (backend, port 5000), `admin.almadina.site` → the admin panel's static `dist/` build.
- **SSL** via Let's Encrypt, auto-renewing.
- Secrets live **only on the server** (not committed to git): `web/.env.local` has `NEXT_PUBLIC_API_URL=https://api.almadina.site/api`; `backend/.env` has `DATABASE_URL`, `JWT_SECRET`, `ADMIN_SECRET_KEY`.

All in-repo hardcoded backend URLs now point at `https://api.almadina.site` (previously the LAN IP `http://192.168.18.18:5000` in mobile/backend/web, and `http://localhost:5000` in `mc-admin-panel/`). The three hardcoded IP references this file used to warn about are fixed — `backend/routes/admin.js` (upload URL) and `web/src/utils/api.ts` now use the production host. **`mobile/src/utils/api.js` auto-detects the dev host from the Expo/Metro connection for local development, but still needs the production URL (`https://api.almadina.site/api`) set before building the mobile app for the Play Store.**

## Development Commands

### Mobile app (`mobile/`)
```bash
cd mobile
npm install
npx expo start
npm run android
npm run ios
npm run web
npm run lint

Backend (backend/)

cd backend
npm install
npm run dev

Requires a backend/.env (see Environment Variables below) and a reachable
PostgreSQL instance. There is no test script and no seed/migration tooling —
schema is created/altered at boot via sequelize.sync({ alter: true }).

Web app (web/)

cd web
npm install
npm run dev
npm run build
npm run start
npm run lint

Optional NEXT_PUBLIC_API_URL env var overrides the default backend URL (now
https://api.almadina.site/api). In production, web/.env.local sets this to
https://api.almadina.site/api (server-only, not committed).

Admin panel (mc-admin-panel/)

cd mc-admin-panel
npm install
npm run dev
npm run build
npm run preview
npm run lint

No env vars — the backend base URL is hardcoded to https://api.almadina.site
throughout (see Cross-Cutting Gotchas). No proxy config in vite.config.js;
relies on the backend's open CORS (origin: '*'). The panel is served in
production from admin.almadina.site (static dist/ build) and talks to
https://api.almadina.site. (Previously this was hardcoded to
http://localhost:5000, so for local dev against a local backend you'd
temporarily point these at your local server.)

Architecture

Backend (backend/)

  - ES modules, single entry point backend/server.js: wires Express, CORS
    (currently origin: '*', wide open), JSON body parsing, /uploads static file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MBN4/madina-collar](https://github.com/MBN4/madina-collar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
