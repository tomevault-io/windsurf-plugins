---
trigger: always_on
description: Generates and maintains Google Sheets that staff use to enter camper activity preferences.
---

# Camp Starfish — CLAUDE.md

A comprehensive reference for working in this repository. Read this before making changes.

---

## 1. What This App Is

**Camp Starfish Photo & Scheduling App** is a web application built by **Hack4Impact-UMD** for Camp Starfish staff, photographers, parents, and administrators. It serves two primary purposes:

1. **Photo Management** — Photographers upload albums of camp photos. Parents view albums of their child's session. Admins moderate flagged photos via an album-item reporting system.
2. **Scheduling** — Camp staff/admins generate per-session schedules across multiple "section" types (BUNDLE, BUNK-JAMBO, NON-BUNK-JAMBO, COMMON), assign program areas/activities, manage night schedules, freeplays, and bunks, and export PDFs for daily use.

The app is **role-gated**: ADMIN, STAFF, PHOTOGRAPHER, PARENT, CAMPER. Different home pages render based on the authenticated user's role (driven by Firebase Auth custom claims set in a Cloud Function).

Repository: `Hack4Impact-UMD/camp-starfish`. Production target: Firebase Hosting (`us-central1`), project id `camp-starfish`.

---

## 2. Tech Stack

### Frontend (root `package.json`)
- **Framework**: Next.js `^15.5.9` (App Router, Turbopack dev)
- **React**: `^19.2.0`
- **Language**: TypeScript `^5.9.3` (target `ES2024`, `@/*` → `./src/*`)
- **UI Components**: Mantine `^8.3.18` (core, form, hooks, modals, notifications, carousel, dates, dropzone)
- **Styling**: Tailwind CSS `^4.2.1` via `@tailwindcss/postcss` + `tailwind-preset-mantine` (theme bridge)
- **Data Fetching**: TanStack Query `^5.90.6`
- **Forms**: TanStack Form `^1.27.7`
- **Tables**: TanStack Table `^8.21.3`
- **Validation**: Zod `^3.25.76`
- **Firebase Client SDK**: `firebase ^11.10.0`
- **PDF**: `@react-pdf/renderer ^4.3.1`
- **Icons**: `react-icons/md` - Material Design icons
- **Dates**: `dayjs`, `moment` (both present — see issues)
- **Other**: `uuid`, `embla-carousel-react`, `classnames`, `cookie`, `mime-types`
- **Devtools** (dev-only): `@tanstack/react-devtools`, `react-query-devtools`, `react-form-devtools`, `react-table-devtools`, `@faker-js/faker`

### Backend — Cloud Functions (`functions/package.json`)
- **Runtime**: Node.js `22`
- **Firebase Functions**: `^6.6.0` (Gen 2 / v2 triggers)
- **Firebase Admin**: `^13.5.0`
- **Google APIs**: `googleapis ^160.0.0`, `google-auth-library ^10.5.0` (Drive integration)
- **Validation**: Zod `^3.25.76`
- **Build**: `tsc && tsc-alias` (resolves `@/*` aliases at build time)
- **Lint**: `eslint-config-google` + typescript-eslint

### Google Apps Script (`apps-script/package.json`)
- **Build**: `esbuild ^0.25.6` (bundles to `dist/` as ESM, ES6 target)
- **Deploy**: `clasp push --force`
- **Purpose**: Generates Google Sheets for camper activity preferences

### Workspace
Root `package.json` declares npm workspaces: `functions`, `apps-script`. The three projects share a single `node_modules` at the root.

---

## 3. Repository Layout

```
camp-starfish/
├── apps-script/           # Google Apps Script (Drive/Sheets integration)
│   └── src/
│       ├── features/preferencesSheets/   # Preference sheet generation
│       ├── utils/properties.ts
│       ├── appsscript.json               # GAS manifest
│       └── globals.d.ts
├── docs/headshots/        # Team headshot PNGs
├── functions/             # Firebase Cloud Functions
│   └── src/
│       ├── config/firebaseAdminConfig.ts
│       ├── data/                         # Admin Firestore + Storage ops
│       ├── features/                     # Function business logic
│       ├── types/
│       ├── index.ts                      # Function exports
│       └── serverUtils.ts
├── public/                # Static assets, font files
├── scripts/
│   ├── generate-emulator-data.ts         # Faker-based seed
│   └── generate-theme-override.ts        # Mantine → Tailwind tokens
├── src/                   # Next.js frontend (see §4)
├── test/emulatorData/     # Saved emulator state (auth, firestore, storage)
├── .firebaserc            # Project: camp-starfish
├── .vscode/
├── eslint.config.ts
├── firebase.json
├── firestore.indexes.json # Currently empty
├── firestore.rules
├── next.config.ts
├── package.json
├── postcss.config.mjs
├── README.md
├── storage.rules          # ⚠️ Currently wide-open (see §11)
├── tsconfig.json
└── tsconfig.tsbuildinfo
```

---

## 4. `src/` — Frontend Detail

### `src/app/` (Next.js App Router)
| Route | File | Purpose | Auth |
|---|---|---|---|
| `/` | `page.tsx` → `LoginPage` / `EmployeeHomePage` / `ParentHomePage` | Role-routed home | none (renders by role) |
| `/albums` | `albums/page.tsx` → `AlbumsPage.tsx` | Album listing (paginated; PR #223) | ADMIN, PHOTOGRAPHER, STAFF, PARENT |
| `/albums/[albumId]` | `albums/[albumId]/page.tsx` → `AlbumPage.tsx` | Single-album view | same |
| `/sessions` | `sessions/page.tsx` | Sessions listing | STAFF/ADMIN |
| `/sessions/[sessionId]` | `SessionPage.tsx` + `SessionCalendar.tsx` | Session detail w/ calendar | STAFF/ADMIN |
| `/sessions/[sessionId]/[sectionId]` | `[sectionId]/page.tsx` | Section detail | STAFF/ADMIN |
| `/demo/program-area-grid` | `demo/program-area-grid/page.tsx` | Internal demo | none |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hack4Impact-UMD/camp-starfish](https://github.com/Hack4Impact-UMD/camp-starfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
