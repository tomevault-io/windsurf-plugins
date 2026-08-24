---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal finance tracker application built with Next.js 16, using the App Router architecture. The stack includes:

- **Framework**: Next.js 16.0.10 (React 19.2.3)
- **Language**: TypeScript with strict mode enabled
- **Database**: Turso (libSQL) via `@libsql/client`
- **Styling**: Tailwind CSS v4 (using new PostCSS plugin architecture)
- **Package Manager**: pnpm 10.24.0
- **Fonts**: Geist Sans and Geist Mono (via next/font)
- **Charts**: Recharts for data visualization

## Development Commands

```bash
# Install dependencies
pnpm install

# Run development server (http://localhost:3000)
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Lint code
pnpm lint

# Database management
pnpm migrate       # Run database migration
pnpm db:test       # Test database connection and verify tables
```

## Application Features

### Core Features

- **Expenses**: Track daily expenses with categories, descriptions, and dual currency (USD/Toman)
- **Income**: Track monthly income by type (salary, freelance, investment, gift, other)
- **Assets**: Track wealth portfolio across 7 categories (cash, crypto, commodity, vehicle, property, bank, investment)
- **Dashboard**: Overview with financial summaries, income vs expenses charts, asset distribution
- **Reports**: Spending analysis with charts and heatmaps
- **Exchange Rate**: Live USD/Toman exchange rate integration
- **Transaction Details Modal**: Click any transaction row to view full details in a modal popup
- **Tag Management**: Hybrid tag system with inline quick actions and centralized Settings management
- **Repeating Expenses**: an expense can repeat on a schedule, on either calendar

### Repeating Expenses

**Repetition is a property of the expense's date — not a separate thing the user manages.** This
follows Todoist's model, and the UI is deliberately just one dropdown (`RepeatField`) sitting under
the date in the expense form: _doesn't repeat / every day / week / month / year / custom…_. Custom
reveals interval, calendar and end date.

There is **no** rules list, no Recurring tab, no upcoming strip and no pause. All of those were
built and then removed: they spent page chrome on a two-way switch and split one mental model — "my
expenses" — across two modes. Stopping a repeat means opening the expense and choosing "doesn't
repeat". Don't reintroduce a separate management surface without a deliberate decision.

Underneath, a `recurringExpenses` row still exists — the user just never sees it. It's created,
updated and deleted purely as a side-effect of saving the expense that owns it
(`syncExpenseRepeat`), and the owning expense's `date` is the rule's `anchorDate`.

**Scheduling model — anchor + index, never a rolling pointer.** A rule stores `anchorDate` and
`postedCount`; every due date is recomputed as `occurrenceAt(anchor, postedCount)`. Stepping
forward from the previous occurrence instead would make the month-end clamp lossy — Jan 31 → Feb 28
→ Mar 28, drifting off the 31st permanently. Recomputing gives Jan 31 → Feb 28 → **Mar 31**.
`nextDueDate` is a denormalized copy of that computation, kept only so "is anything due?" is one
indexed query. **Never increment it in place.**

**Per-repeat calendar.** `calendar` is `gregorian` or `jalali` and is meaningful only for
monthly/yearly — a week is seven days in both systems, so the control is hidden for daily/weekly.
Rent on 1 Farvardin genuinely recurs on Jalali months; a USD subscription recurs on Gregorian.
Defaults to the user's resolved calendar.

**Where the code lives:**

- `src/core/recurring/schedule.ts` — pure occurrence math, no clock or I/O, so the server
  materializer and the form's live preview can't drift apart. Well covered by tests; extend them
  when touching it.
- `src/core/recurring/materialize.ts` — turns due rules into expense rows.
- `src/core/database/expense-repeat.ts` — creates/updates/deletes the rule behind an expense.
- `src/features/expenses/components/RepeatField/` — the entire UI.

**Materialization** runs from two places: the daily cron (`/api/cron/reports`, all users) and
lazily on the read path (`GET /api/expenses`, current user only) so someone opening the app before
the cron fires still sees today's rent. They can race safely — the partial unique index
`idxExpenseRecurringOnce` makes a duplicate insert a no-op.

**Rules that must hold:**

- "Today" comes from `todayInTimeZone()` (Asia/Tehran), never UTC. Expense dates carry no timezone
  and the cron runs at 09:00 UTC, so a UTC "today" posts 1st-of-month rules on the wrong local day.
- Catch-up occurrences snapshot the rate that was current **on their own due date**
  (`getEntryRateOn`), not today's — a three-month catch-up must stay historically honest.
- A missing rate breaks the loop **without** advancing `postedCount`, so the occurrence is retried
  rather than silently skipped.
- Removing a repeat keeps the expenses it already posted (`expenses.recurringId` is
  `ON DELETE SET NULL`) — that's money actually spent.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erfanansari/kharji](https://github.com/erfanansari/kharji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
