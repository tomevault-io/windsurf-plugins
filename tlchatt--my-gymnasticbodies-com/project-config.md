---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run start    # Dev server — proxies to https://api.gymnasticbodies.com by default
npm run build    # Production build (CRA)
npm run test     # Jest (CRA defaults, no custom config)
```

Single test: `npm run test -- --testPathPattern=<filename>` or `npm run test -- --watch`.

## Architecture

React 17 CRA single-page app. Redux + redux-thunk for all async state. Material-UI v4 throughout.

### Directory layout

| Path | Purpose |
|---|---|
| `src/Store/Action/` | All async thunks and action creators |
| `src/Store/Reducers/` | Redux reducers |
| `src/Store/util.js` | `updateObject`, `getCurrentWeek`, `AxiosConfig` |
| `src/Containers/` | Page-level components (routed) |
| `src/Components/` | Reusable UI components |
| `src/data/` | Large static JS workout data files (1–6 MB each) |
| `src/HOC/firebase.js` | Firebase Realtime DB init (maintenance/refresh signals only) |

### Redux store shape

```
{
  login:        { auth, webToken, firstName, lastName, UserId, timezone,
                  userLevel, levelId, isFreeMember, isAllAccessUser,
                  isThriveUser, isAdmin, integratedPlans }
  calendar:     { schedule, toasts, success/fail flags }
  classes:      { all available classes }
  data:         { allData }
  subClasses:   { }
  legacyCourse: { }
  demoModal:    { }
  freeMember:   { }
  levels:       { user progression }
  buildYourOwn: { BYO workout state }
  OhNo:         { error modal state }
  OpenDrawer:   { drawer visibility }
}
```

### Authentication

Login POSTs to `/api/authentication` (new) or `/auth` (legacy). Response contains `jwtAuthorizationToken` + `jwtRefreshToken`, both stored in `localStorage` with expiration timestamps. `checkAuthTimeout` schedules auto-logout. Token is decoded via `jsonwebtoken` to extract user info.

`AxiosConfig` (in `util.js`) builds the `Authorization: Bearer <token>` header — always use this for authenticated requests rather than building headers manually.

**Renewal / paywall flow:** The `LoginNew` thunk calls `GET /api/user/renewalStatus?email=...` after credentials are verified. If `needsRenewal: true` (user's `migration_type` is `active_expired`), the browser is redirected to `https://app.gymnasticbodies.com/renew?email=...` before login is dispatched. On successful re-subscription the user is sent back with an auth token.

**User migration types** (`migration_type` field on the server-side `user` table): `stripe`, `auth_net_subscriber`, `active_current`, `active_expired`, `inactive`.

The renewal redirect in `LoginNew` is live — `needsRenewal: true` redirects to `https://app.gymnasticbodies.com/renew?email=...` before dispatching login.

### API endpoints

Two concurrent base URLs are in use during an ongoing migration:

| Variable | URL | Usage |
|---|---|---|
| `REACT_APP_API` | `https://api.gymnasticbodies.com` | Legacy AWS — auth, schedule, BYO |
| `REACT_APP_API_NEW` | `https://gymnasticbodies-com.vercel.app` | Neon/app.gymnasticbodies.com — new endpoints |

New feature work should target `REACT_APP_API_NEW`. The legacy API remains for schedule, BYO workouts, and token refresh.

### Routing

`App.js` has two authenticated route trees — **you must add new routes to both or they won't work for all users:**

| Condition | Route tree | Component |
|---|---|---|
| `showAllAccessSite && isAuth` | Route 2 | `<NewMemberSite>` — has its own inner `<Switch>` |
| `isAuth` (regular) | Route 3 | Full flat route list with Header + Footer |

`NewMemberSite` (`src/Containers/NewMemberSite/index.jsx`) renders its own `<Switch>` internally. Routes defined only in Route 3 will 404→redirect to `/` for all-access users. Add new routes to `NewMemberSite`'s inner Switch **and** to Route 3 in `App.js`.

Notable routes (both trees): `/course-library`, `/class-finder`, `/class-finder/:category`, `/my-courses`, `/eqiupment-list`, `/information`, `/advocates`.

### Static workout data

`src/data/` holds large pre-built JS objects (not API-fetched). `AllDataForWorkout.js` (1.5 MB) and `programCoreData.js` (1.6 MB) are the primary sources. These are imported directly and hydrated into the Redux `data` slice.

### Firebase usage

Firebase Realtime DB is used exclusively for maintenance-mode flags and force-refresh signals. It is **not** used for auth or user data storage in this app (auth is JWT-based).

## Deployment

**Manual deploy (preferred):** `bash claudeTools/deploy.sh` — builds production, syncs to S3, invalidates CloudFront in one shot.

Bitbucket Pipelines → S3 + CloudFront (legacy pipeline, still wired up).

| Branch | S3 bucket | CloudFront |
|---|---|---|
| `master` | `my.react2026` | `E2TAHYRIUSC1ZN` (`my.gymnasticbodies.com`) |
| `Develop` | `my.react-testing` | `E1KQMIVMY2A66G` |
| `Staging` | `my.internal-testing` | `E2NDG89QP09SYX` |

**Important — `my.react2026` bucket:** ACLs are disabled on this bucket (Object Ownership = Bucket owner enforced). Do **not** use `--acl public-read` when syncing — it will fail with `AccessControlListNotSupported`. Public access is granted via bucket policy, not ACLs. The deploy script already handles this correctly.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tlchatt/my.gymnasticbodies.com](https://github.com/tlchatt/my.gymnasticbodies.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
