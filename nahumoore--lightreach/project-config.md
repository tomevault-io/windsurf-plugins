---
trigger: always_on
description: > ⚠️ **This is NOT the Next.js you know.**
---

# Lightreach — Dev Handbook

> ⚠️ **This is NOT the Next.js you know.**
> Next.js 16 has breaking changes. Before writing any app code read the relevant guide
> in `apps/web/node_modules/next/dist/docs/`. Heed deprecation notices.

---

## What is Lightreach?

Lightreach is a **free, self-hosted, lightweight cold-email outreach platform** — a slim
alternative to Instantly / Smartlead / lemlist. It is single-user, requires no external
services beyond your own SMTP mailboxes, and runs from a single `pnpm dev` command.

### Feature Map

| Section | What it does |
|---|---|
| **Dashboard** | Activity overview and send analytics. |
| **Connections** | Add SMTP mailboxes (Gmail, Outlook, custom). Test, pause, track daily limits. Optional IMAP config per mailbox. |
| **Inbox / Emails** | Poll received mail across all IMAP mailboxes. Automatic reply and warmup detection. |
| **Leads** | Upload CSVs with column-mapping wizard. Manage lists + individual leads. |
| **Sequences** | Multi-step email sequences with configurable `delayDays` between steps. Write with `{spintax\|options}` and `{{variable\|fallback}}` placeholders. Live preview. |
| **Campaigns** | Pair a sequence with a lead list, assign mailboxes to rotate across, set schedule. |
| **Scheduling** | Send-window (time of day), days of week, daily cap, min/max delay jitter between sends. |
| **Settings** | App-level defaults, encryption key status, sending behavior. |

---

## Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 16 (App Router) |
| UI library | shadcn/ui (radix-rhea style, Tabler icons) |
| Styling | Tailwind CSS v4 (CSS-first config) |
| Component pkg | `@workspace/ui` — shared across all apps |
| Data | SQLite via Drizzle ORM + drizzle-kit |
| Email (send) | Nodemailer (user-provided SMTP credentials) |
| Email (receive) | imapflow + mailparser (IMAP polling, reply/warmup detection) |
| Monorepo | pnpm workspaces + Turborepo |
| Language | TypeScript 5 strict |

---

## Repo Layout

```
lightreach/
├── apps/
│   └── web/              # Next.js 16 app (the main UI + API)
├── packages/
│   ├── core/             # @workspace/core — pure business logic
│   │   ├── src/spintax.ts      # {a|b|c} expansion
│   │   ├── src/variables.ts    # {{firstName|fallback}} rendering
│   │   ├── src/crypto.ts       # AES-256-GCM encrypt/decrypt for SMTP secrets
│   │   ├── src/csv.ts          # CSV parsing + header-mapping helpers
│   │   ├── src/rotation.ts     # round-robin mailbox rotation
│   │   ├── src/email/transport.ts  # nodemailer transport builder + sendMail
│   │   └── src/email/imap.ts       # imapflow IMAP polling + reply detection
│   ├── db/               # @workspace/db — Drizzle schema + client
│   │   ├── src/schema.ts
│   │   ├── src/client.ts
│   │   └── drizzle.config.ts
│   ├── ui/               # @workspace/ui — shadcn components + global CSS
│   ├── eslint-config/
│   └── typescript-config/
```

---

## Commands

```bash
# Development
pnpm dev          # run all apps in dev mode (turbo)
pnpm build        # production build
pnpm lint         # lint all workspaces
pnpm typecheck    # type-check all workspaces
pnpm format       # prettier all workspaces

# Database (run from repo root)
pnpm db:generate  # drizzle-kit generate — regenerate SQL migrations from schema.ts
pnpm db:migrate   # drizzle-kit migrate — apply migrations to data.db
pnpm db:studio    # open Drizzle Studio to browse data

# Adding shadcn components (always run from repo root)
pnpm dlx shadcn@latest add <component> -c apps/web
# → places the component in packages/ui/src/components/
```

---

## Code Conventions

### Imports
```ts
import { Button } from "@workspace/ui/components/button";   // UI components
import { db }     from "@workspace/db";                      // database client
import { expandSpintax } from "@workspace/core/spintax";    // core utilities
```

### Mutations → Server Actions
Prefer React Server Actions (`'use server'`) for all DB writes.
Define actions in `apps/web/app/<feature>/actions.ts`.

### Database access
Only import `@workspace/db` from **server** code (Server Components, Route Handlers,
Server Actions, `instrumentation.ts`). Never import it from `'use client'` files.

### Server Actions pattern
```ts
// app/connections/actions.ts
'use server'
import { db } from '@workspace/db'
import { connections } from '@workspace/db/schema'
import { revalidatePath } from 'next/cache'

export async function createConnection(data: NewConnection) {
  await db.insert(connections).values(data)
  revalidatePath('/connections')
}
```

---

## Theme & Styling

**Dark-first, blue primary.** The app defaults to dark mode.

- Use **semantic tokens only** (`bg-background`, `text-foreground`, `text-primary`,
  `border-border`, …). **Never hard-code hex or hsl values.**
- Primary blue lives in `--primary` (oklch). The dark-mode value is the showcase.
- All tokens are defined in `packages/ui/src/styles/globals.css`.
- Toggle theme with the `d` key (or the header toggle button).

---

## Security Rules

- **SMTP passwords are encrypted at rest** using AES-256-GCM before being stored in
  SQLite. Use `encrypt()`/`decrypt()` from `@workspace/core/crypto`.
- `APP_ENCRYPTION_KEY` is a required env var (32-byte hex string). The app should
  warn on startup if it is missing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nahumoore/lightreach](https://github.com/nahumoore/lightreach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
