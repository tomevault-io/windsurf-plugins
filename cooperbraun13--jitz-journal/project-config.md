---
trigger: always_on
description: **Next.js (App Router) + PostgreSQL + Prisma + NextAuth.js**
---

# BJJ Training Tracker — Project Plan

## Recommended Stack

**Next.js (App Router) + PostgreSQL + Prisma + NextAuth.js**

**Rationale:**
- Next.js handles both frontend and backend in a single project, reducing operational complexity.
- Prisma provides type-safe database access and schema migrations for PostgreSQL.
- NextAuth.js handles multi-user authentication with minimal boilerplate and has first-class Next.js support.
- All are production-grade, well-maintained, and have strong community support.

---

## Architecture Overview

```
Browser (React / Next.js)
       |
  Next.js App Router
  ├── /app               → Pages & layouts
  ├── /app/api           → API Route Handlers (REST endpoints)
  ├── /lib               → Prisma client, auth config, utilities
  └── /components        → Shared UI components
       |
  Prisma ORM
       |
  PostgreSQL
```

---

## UI Design Direction

The UI will be built to a high visual standard — not a generic dashboard.

**Key principles:**
- **Design system:** shadcn/ui as the component base, extended with custom styling
- **Styling:** Tailwind CSS with a custom theme (colors, typography, spacing)
- **Charts:** Recharts with custom-styled components matching the theme
- **Motion:** Framer Motion for page transitions and micro-interactions
- **Typography:** Clean hierarchy using a single variable font (Inter or Geist)
- **Layout:** Sidebar navigation on desktop, bottom tab bar on mobile
- **Light/Dark toggle:** System preference as default, user-overridable, persisted to their profile

### Color Palette

| Role | Light | Dark |
|---|---|---|
| Background | `#F8F8F6` | `#0F0F0F` |
| Surface | `#FFFFFF` | `#1A1A1A` |
| Primary | `#C41E3A` (BJJ red) | `#E8294F` |
| Primary foreground | `#FFFFFF` | `#FFFFFF` |
| Muted | `#F1F1EF` | `#242424` |
| Border | `#E4E4E0` | `#2E2E2E` |
| Text primary | `#111111` | `#F2F2F2` |
| Text muted | `#6B6B6B` | `#888888` |

Belt colors render as actual colored badges (white, blue, purple, brown, black) with stripe pip indicators.

### Belt Badge Visual

```
[ BLUE ●●●○○ ]    → Blue belt, 3 stripes
[ PURPLE ●●●●○ ]  → Purple belt, 4 stripes (next = brown)
```

---

## Authentication

### Providers

| Provider | Notes |
|---|---|
| Google OAuth | Standard NextAuth.js provider |
| GitHub OAuth | Standard NextAuth.js provider |
| Email magic link | NextAuth.js Email provider — requires SMTP service (e.g. Resend or SendGrid) |
| Username & password | NextAuth.js Credentials provider + bcrypt hashing |

### Auth Flow

```
/auth/signin
  ├── "Continue with Google"
  ├── "Continue with GitHub"
  ├── "Send magic link" (email input)
  └── "Sign in with email & password"

New user (any provider)
  └── → /onboarding  (profile completion required)

Returning user (onboardingComplete = true)
  └── → /dashboard
```

Password requirements (credentials provider): minimum 12 characters, at least one uppercase, one number, one special character. Enforced with Zod on both client and server.

---

## Onboarding Flow

Until `onboardingComplete = true` is set on the User record, all routes redirect to `/onboarding` via Next.js middleware.

**Step 1 — Create account** (handled by NextAuth.js)

**Step 2 — Complete profile** (required before accessing the app)

### Onboarding Fields

| Field | Type | Required |
|---|---|---|
| displayName | String | Yes |
| dateOfBirth | Date | No |
| heightCm | Decimal | Yes |
| weightKg | Decimal | Yes |
| belt | Enum | Yes |
| stripes | Int (0–4) | Yes |
| gym | String | Yes |
| city | String | No |
| country | String | No |
| trainingStartDate | Date | No |
| preferredTheme | Enum (Light / Dark / System) | Yes |
| avatar | String (URL) | No |

---

## Database Schema

### User

```prisma
model User {
  id                  String    @id @default(uuid())
  email               String    @unique
  passwordHash        String?
  displayName         String
  avatar              String?
  dateOfBirth         DateTime?
  heightCm            Decimal
  weightKg            Decimal
  belt                Belt
  stripes             Int       @default(0) // 0–4
  gym                 String
  city                String?
  country             String?
  trainingStartDate   DateTime?
  preferredTheme      Theme     @default(SYSTEM)
  onboardingComplete  Boolean   @default(false)
  createdAt           DateTime  @default(now())
  updatedAt           DateTime  @updatedAt
}

enum Belt {
  WHITE
  BLUE
  PURPLE
  BROWN
  BLACK
}

enum Theme {
  LIGHT
  DARK
  SYSTEM
}
```

### Belt & Stripe Rules

- Each belt has stripes 0–4 (5 stripes per belt before promotion)
- Stripe 4 = one away from promotion to the next belt
- Black belt is the terminal belt; stripes continue up to degree 6 (coral/red belt can be added later)
- Belt and stripe are displayed everywhere the user appears

### Training Sessions

```prisma
model Session {
  id              String        @id @default(uuid())
  userId          String
  date            DateTime
  durationMinutes Int
  type            SessionType
  gym             String?
  notes           String?
  createdAt       DateTime      @default(now())
  user            User          @relation(fields: [userId], references: [id])
  techniques      Technique[]
  sparringRounds  SparringRound[]
}

enum SessionType {
  GI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cooperbraun13/jitz-journal](https://github.com/cooperbraun13/jitz-journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
