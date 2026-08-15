---
trigger: always_on
description: - Prioritize security, maintainability, and least-privilege by default.
---

# AGENTS.md

## Core Principles
- Prioritize security, maintainability, and least-privilege by default.
- Never ship code that violates these rules unless explicitly instructed otherwise.

## UI / UX (forms & screens)

When designing or changing any screen, treat **density, alignment, and input affordances** as first-class requirements—not polish after the fact.

### Density & vertical rhythm
- Prefer a **medium-compact** layout: enough breathing room to scan and click, without tall empty bands, oversized section padding, or stacked whitespace that forces unnecessary scrolling.
- Tighten vertical spacing where content is related (label → control → hint); keep clearer separation between distinct sections.
- Avoid one-off “spacious” blocks that break the density of surrounding Admin / app chrome.

### Alignment & field width
- Align labels and controls consistently within a form or grid (shared columns, matching baselines, predictable gutters).
- Size inputs to the **expected content length**—short values (ports, days, codes, prefixes, ticket IDs) should use compact widths, not full-bleed `w-full` when that wastes space or weakens the visual hierarchy.
- Prefer grids or fixed/`max-w-*` controls for short fields; reserve full width for prose, descriptions, URLs that may be long, and rich text.

### Dates
- Calendar dates are always **`YYYY-MM-DD`** in the UI and when bound to form controls.
- Use a native **date picker** (`<input type="date">`) so users can pick from a calendar **or type freely**; do not force spinner-style increment/decrement UX.
- Do **not** use `type="number"` (or other spinbutton controls) for calendar dates.
- Follow the project date pipeline (API `YYYY-MM-DD` / `::text`, normalize before binding, local-date display)—see `.cursor/rules/easy-kanban-dates.mdc` and `src/utils/dateUtils.ts`.

### Numeric fields (non-date)
- For quantities and settings that are numbers, allow free typing; hide browser spinner arrows where the project already does (e.g. `ADMIN_NUMERIC_INPUT_CLASS` in `src/utils/adminFieldLimits.ts`) so users are not nudged into click-to-increment behavior.

## Documentation Policy
- **DO NOT create new .md documentation files** unless explicitly requested by the user
- **ESPECIALLY for QA/testing work**: Do NOT create README files, CHANGES files, or summary documents
- Update existing documentation when making changes to related code
- Use code comments for explaining implementation details
- Reserve documentation files for:
  - Major architectural decisions (when requested)
  - Setup/configuration guides (when requested)
  - API reference documentation (when requested)
- Exception: README.md updates are acceptable for significant feature additions

## Package Management (npm)
- Always choose secure, actively maintained packages.
- Packages currently used in this project:
 - **Backend**: `express`, `pg`, `redis`, `socket.io`, `bcrypt`, `jsonwebtoken`, `multer`, `nodemailer`, `node-cron`, `express-rate-limit`, `cors`, `axios`, `zod`, `exceljs`
 - **Frontend**: `react`, `react-dom`, `react-i18next`, `i18next`, `i18next-browser-languagedetector`, `@tiptap/*` (rich text editor), `@dnd-kit/*` (drag-and-drop), `lucide-react`, `react-joyride`, `react-window`, `recharts`, `exceljs`, `dompurify`, `socket.io-client`
  - **Real-time**: `socket.io`, `socket.io-client`, `@socket.io/redis-adapter`, `redis`
  - **Build/Dev**: `vite`, `typescript`, `tailwindcss`, `eslint`, `concurrently`
- Avoid packages with known vulnerabilities, >1 year without updates, or <1k weekly downloads unless there is a very specific reason.
- Use exact versions or caret ranges (^) for dependencies; versions are pinned via package-lock.json for reproducible builds.

## API Routes & Endpoints
- EVERY route must be authenticated by default using `authenticateToken` middleware.
- Only make a route public if it is explicitly listed as public:
  - **Authentication**: `/api/auth/login`, `/api/auth/activate-account`, `/api/auth/verify-invitation`, `/api/auth/google/*`, `/api/auth/demo-credentials`, `/api/auth/check-*`
  - **Password Reset**: `/api/password-reset/request`, `/api/password-reset/reset`, `/api/password-reset/verify/:token`
  - **Health Checks**: `/health`, `/ready`, `/api/ready`, `/api/version`
  - **Public Settings**: `/api/settings` (GET only, for site name, mail status, OAuth config)
  - **CSP reports**: `/api/csp-report` (POST only — browser Content-Security-Policy violation beacons; rate-limited)
  - **Admin Portal**: `/api/admin-portal/*` (uses `INSTANCE_TOKEN` auth, not user JWT)
- Use existing auth middleware from `server/middleware/auth.js`:
  - `authenticateToken` - JWT token validation (required for all protected routes)
  - `requireRole(['admin'])` - Role-based access control for admin-only endpoints
- Apply rate limiting for sensitive public endpoints (see `server/middleware/rateLimiters.js`):
 - `loginLimiter` for login attempts
 - `passwordResetRequestLimiter` (3/hour) and `passwordResetCompletionLimiter` (6/hour)
 - `registrationLimiter`, `invitationVerifyLimiter` (GET verify), and `activationLimiter` (POST activate) for account creation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drenlia-inc/agila](https://github.com/drenlia-inc/agila) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
