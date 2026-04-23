---
trigger: always_on
description: A full-stack web app for tracking 20+ university applications. Students can manage universities, track requirements (essays, test scores, recommendations), manage deadlines, and view a calendar timeline. Data is fully isolated per user via Clerk authentication.
---

# University Applications Organizer — Claude Context

## What This Project Is
A full-stack web app for tracking 20+ university applications. Students can manage universities, track requirements (essays, test scores, recommendations), manage deadlines, and view a calendar timeline. Data is fully isolated per user via Clerk authentication.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript 5.7 |
| Database ORM | Prisma 6 |
| Database | Neon PostgreSQL (production) |
| Auth | Clerk (`@clerk/nextjs` v7) |
| Styling | Tailwind CSS 3.4 |
| UI Primitives | Radix UI (checkbox, dialog, dropdown, label, select, slot) |
| Icons | Lucide React |
| Date Picker | react-day-picker |
| Utilities | clsx, tailwind-merge, class-variance-authority |
| Runtime | Node.js 18.17+ |

---

## Features Implemented

### Authentication (Clerk)
- Sign in / Sign up at `/sign-in` and `/sign-up` (embedded Clerk components)
- OAuth providers: Google, Apple, Microsoft (Outlook/Azure AD), Email/Password
- Middleware protects all routes except `/sign-in` and `/sign-up`
- Custom nav user menu: orange circle with initials, dropdown shows name/email + sign-out
- Per-user data isolation: every Prisma query is scoped by `userId` from `auth()`

### Dashboard (`/`)
- Stats overview: total universities, count by status (considering/applied/accepted/waitlisted/rejected/enrolled)
- Upcoming deadlines widget with 7/14/30-day filter tabs
- Quick actions: Add University, View All Universities, Timeline
- Status cards are clickable (links to universities page filtered by that status)

### Universities (`/universities`)
- Card grid with color-coded top stripe by category (reach=red, target=blue, safety=green)
- Status and category badges on each card
- Next deadline + requirements progress bar per card
- Search by name, filter by status and category
- Add/Edit modal form (full-screen, all fields)
- University search: US College Scorecard API + Hipolabs worldwide API (auto-fills name, location, website)

### University Detail (`/universities/[id]`)
- Edit status and category inline
- Requirements checklist: add/edit/delete, toggle completion, progress bar
- Deadlines manager: add/edit/delete, toggle completion, color-coded urgency (red/orange/blue/green)
- Notes section: view/edit toggle, character count

### Timeline (`/timeline`)
- Monthly calendar view of all deadlines
- Color-coded urgency: red (urgent ≤3 days), orange (4–7 days), blue (normal)
- Filter by university or deadline type
- Countdown (days until each deadline)

### University Comparison
- Side-by-side comparison of multiple universities
- Key metrics: rankings, acceptance rates, costs, deadlines

---

## Database Schema

### University
```prisma
model University {
  id                    String      @id @default(cuid())
  userId                String                          // Clerk user ID — all queries scoped by this
  name                  String
  location              String
  program               String
  status                String      @default("considering")
  // "considering" | "applied" | "accepted" | "rejected" | "waitlisted" | "enrolled"
  category              String?     // "reach" | "target" | "safety"
  ranking               Int?
  acceptanceRate        Float?
  websiteUrl            String?
  notes                 String?
  researchNotes         String?
  applicationDeadline   DateTime?
  earlyDeadline         DateTime?
  decisionDate          DateTime?
  tuition               Float?
  applicationFee        Float?
  estimatedCostOfLiving Float?
  financialAidDeadline  DateTime?
  scholarshipNotes      String?
  createdAt             DateTime    @default(now())
  updatedAt             DateTime    @updatedAt
  requirements          Requirement[]
  deadlines             Deadline[]
  @@index([userId])
}
```

### Requirement
```prisma
model Requirement {
  id           String    @id @default(cuid())
  universityId String
  type         String    // essay, test_score, recommendation, transcript, portfolio, other
  title        String
  description  String?
  completed    Boolean   @default(false)
  deadline     DateTime?
  notes        String?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  university   University @relation(fields: [universityId], references: [id], onDelete: Cascade)
  @@index([universityId])
}
```

### Deadline
```prisma
model Deadline {
  id           String    @id @default(cuid())
  universityId String
  title        String
  date         DateTime
  type         String    // application, financial_aid, scholarship, decision, deposit, housing, other
  description  String?
  completed    Boolean   @default(false)
  reminderDays Int?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  university   University @relation(fields: [universityId], references: [id], onDelete: Cascade)
  @@index([universityId])
  @@index([date])
}
```

---

## Environment Variables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coderAshraful) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
