---
trigger: always_on
description: Living handoff for this repo. Update this file as modernization work progresses.
---

# AGENTS.md

Living handoff for this repo. Update this file as modernization work progresses.

## 1) Project Snapshot

- App: `Wallet Watch` (modernized MERN + Plaid integration)
- Goal: modernize codebase and redeploy on a free/low-cost stack for limited users
- Current shape: split frontend/backend with independent `package.json` files
- Primary branch: `master`
- Production deployment status (as of March 2, 2026):
  - Frontend: `https://wallet-watch-seven.vercel.app`
  - Backend: `https://wallet-watch-t5cu.onrender.com`
  - Database: MongoDB Atlas `M0`

## 2) Repo Walkthrough

### Root

- `README.md`: local setup + deployment/redeploy/rollback/troubleshooting docs
- `.gitignore`: now ignores dependency folders/env/build artifacts across root + backend + frontend
- `.netlify/state.json`: Netlify site metadata
- `backend/`: Express/Mongoose API
- `frontend/`: Vite React app

### Backend (`backend/`)

- Entrypoint: `server.js`
  - Loads env via `dotenv`
  - Configures `express.json()`, `cors()`, request logging middleware
  - Mounts routes:
    - `/api/user`
    - `/api/transactions`
    - `/api/plaid`
  - Connects via `mongoose.connect(process.env.MONGO_URI)` then `app.listen(process.env.PORT)`

- Routes/controllers/models:
  - `routes/user.js` + `controllers/userController.js`
    - `POST /register`, `POST /login`
    - JWT with `SECRET`, 3-day expiry
  - `routes/transactions.js` + `controllers/transactionController.js`
    - Auth-protected CRUD for transactions
    - Supports `sortBy` query (`newest`, `oldest`, `highest`, etc.)
  - `routes/plaid.js` + `controllers/plaidController.js`
    - Auth-protected Plaid Link token creation, token exchange, transaction sync
  - `middleware/requireAuth.js`
    - Expects `Authorization: Bearer <token>`
  - `models/user.js`
    - Fields: `username`, `password`, `plaidItems[]`, plus legacy migration fields `access_token`, `item_id[]`, `plaidCursor`
  - `models/transaction.js`
    - Fields: `name`, `type`, `amount`, `userID`, `plaidTransactionID`, `date`, `category[]`

- Deployment artifact:
  - `Procfile` (`web: node server.js`) indicates former Heroku deploy model

- Environment variables in use:
  - `PORT`
  - `MONGO_URI`
  - `SECRET`
  - `FRONTEND_ORIGIN`
  - `RATE_LIMIT_WINDOW_MS`
  - `RATE_LIMIT_MAX`
  - `PLAID_CLIENT_ID`
  - `PLAID_SECRET`
  - `PLAID_ENV`
  - `PLAID_PRODUCTS`
  - `PLAID_COUNTRY_CODES`

### Frontend (`frontend/`)

- Framework/tooling: Vite + React
- Entrypoint: `src/main.jsx` with `AuthContextProvider` + `TransactionContextProvider`
- Routing: `src/App.jsx` using React Router v6
  - Protected routes for app pages, login/register gating by auth context
- State:
  - `context/authContext.jsx`: auth reducer with `LOGIN`/`LOGOUT`
  - `context/transactionContext.jsx`: transaction reducer
- Pages:
  - `Home`: transactions list + sort + pagination + sync button
  - `LinkAccounts`: Plaid Link flow
  - `Login`, `Register`, `Info`, `Income`, `Spending` (all exposed in navigation)
- API calls:
  - Centralized helper in `src/lib/api.js`
  - Uses `VITE_API_BASE_URL` for explicit backend origin
- UI system:
  - Global design tokens + responsive layout in `src/index.css`
  - Shared finance helpers in `src/lib/finance.js`
- Current hosting config:
  - Vercel is the active frontend host
  - `frontend/netlify.toml` remains available for Netlify fallback deploys

## 3) Current Risks / Technical Debt (Observed)

### Repo hygiene and operability

- Legacy history still contains large old commits (current tip is clean)
- No nested git repo remains under `backend/wallet-watch/.git`

### Backend correctness/security

- Legacy Plaid fields (`access_token`, `item_id`, `plaidCursor`) are retained for backward compatibility.
- Active Plaid sync now uses `plaidItems[]` (`itemId`, `accessToken`, `cursor`) with per-item cursors.

### Frontend architecture/UX

- Component-level tests are still sparse on key auth/transaction UI flows
- `useLinkAccount.js` exists but is empty/unused
- There is still duplicated list-fetch logic across `Home`, `Income`, and `Spending`

### Deployment model drift

- Vercel preview URLs can cause CORS failures unless explicitly added to `FRONTEND_ORIGIN`
- Render free tier cold starts can make first API request slow
- Operational runbook exists in `README.md`, but no automated uptime checks yet

## 4) Current Deployment Architecture

- Frontend: Vite React SPA on Vercel (Hobby)
- Backend: Express API on Render Web Service (free tier)
- Database: MongoDB Atlas `M0`
- Auth/API:
  - Keep JWT auth initially
  - Add strict CORS allowlist with frontend domain
  - Use explicit `VITE_API_BASE_URL` env variable in frontend (avoid host-coupled rewrites)

Reason: this keeps hosting costs low while maintaining an explicit frontend/backend boundary.

## 5) Modernization Plan (Phased)

### Phase 0: Safety + Baseline

- [x] Create branch `modernize-phase-0`
- [x] Add/validate `.gitignore` for `backend/node_modules`, `frontend/node_modules`, build outputs
- [x] Remove tracked `backend/node_modules` from git history tip (keep lockfiles)
- [x] Add `README` setup instructions and env template(s)
- [x] Add smoke tests for auth + transaction CRUD API

### Phase 1: Backend hardening


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simon-rupp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
