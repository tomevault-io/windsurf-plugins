---
trigger: always_on
description: You are building **Obelisk**, a Discord-like group chat app where identity comes from Nostr keypairs. No emails, no passwords — cryptographic identity only.
---

# AGENTS.md — Obelisk

You are building **Obelisk**, a Discord-like group chat app where identity comes from Nostr keypairs. No emails, no passwords — cryptographic identity only.

Built for La Crypta's **IDENTITY Hackathon** (April 2026). See [ROADMAP.md](ROADMAP.md) for the full development plan.

## Architecture

Nostr handles **identity & auth** (keys, profiles, NIP-05, signing). The server handles **everything else** (channels, messages, members, roles, permissions, real-time delivery).

```
Frontend          Next.js + Tailwind (La Crypta UI)
Auth              Nostr (NIP-07 / nsec / NIP-46 bunker)
Backend           Next.js API Routes + custom server (server.ts)
Database          PostgreSQL (self-hosted via Docker)
ORM               Prisma 7 + @prisma/adapter-pg
Real-time         Socket.io (via server.ts)
Voice             WebSocket audio relay (via server.ts + Socket.io)
Payments          Nostr Wallet Connect (NIP-47) — src/lib/nwc.ts, src/lib/crypto.ts
Admin CLI         scripts/admin-cli — nsec / NIP-46 bunker auth, AI-agent friendly
```

## Stack
- **Next.js 16** + TypeScript + Tailwind CSS v4
- **NDK** (Nostr Dev Kit v3) — Nostr abstraction for auth & profiles
- **Prisma 7** — ORM with PostgreSQL via `@prisma/adapter-pg`
- **Socket.io** — Real-time messaging (via `server.ts`)
- **WebSocket Audio** — Voice channels via Socket.io audio relay (works through tunnels/proxies, see `docs/voice-system.md`)
- **Zustand** — client-side state management
- **nostr-tools** — low-level Nostr utilities
- **Vitest** + **React Testing Library** — testing

## Project Structure
```
prisma/
├── schema.prisma         # Data model (Server, Channel, Message, Member, etc.)
├── seed.ts               # DB seed script
└── migrations/           # Prisma migrations
server.ts                 # Custom Next.js + Socket.io server
src/
├── app/
│   ├── layout.tsx        # Root layout (La Crypta theme)
│   ├── page.tsx          # Landing / main page
│   ├── chat/page.tsx     # Chat UI (Discord-like layout)
│   ├── admin/page.tsx    # Server administration panel
│   ├── moderation/page.tsx # Moderation dashboard
│   └── api/
│       ├── auth/         # challenge → sign → verify → session
│       ├── channels/     # CRUD channels + messages
│       ├── members/      # Member management
│       ├── admin/        # Server settings, roles, bans, kicks
│       └── moderation/   # Reports, mutes, warnings, mod log
├── components/
│   ├── Navbar.tsx        # Top navigation + user menu
│   ├── LoginModal.tsx    # 3 auth methods + QR bunker flow
│   ├── ObeliskIcon.tsx   # App icon
│   ├── chat/
│   │   ├── ServerBar.tsx     # Server icon sidebar (Discord-like)
│   │   ├── ChannelSidebar.tsx # Channel list sidebar
│   │   ├── MessageArea.tsx    # Message display with scroll
│   │   └── MessageInput.tsx   # Message composer
│   ├── admin/
│   │   ├── MemberRow.tsx     # Member list row with actions
│   │   ├── ConfirmDialog.tsx # Confirmation modal
│   │   └── RoleBadge.tsx     # Role display badge
│   └── moderation/
│       └── ModActionCard.tsx # Moderation action display
├── lib/
│   ├── nostr.ts          # NDK setup, login, relay mgmt
│   ├── auth.ts           # Client-side auth (challenge/verify flow)
│   ├── api-auth.ts       # API route auth helpers
│   ├── backend-auth.ts   # Server-side session verification
│   ├── auth-roles.ts     # Role & permission logic
│   ├── db.ts             # Prisma client singleton
│   └── db-server.ts      # Server initialization helpers
├── store/
│   ├── auth.ts           # Auth state (Zustand + localStorage)
│   ├── chat.ts           # Chat state (channels, messages, socket)
│   └── nav.ts            # Navigation state
├── generated/prisma/     # Generated Prisma client
└── test/
    ├── setup.ts          # Vitest setup
    └── mocks/ndk.ts      # NDK mock for tests
```

## Commands
```bash
npm install          # Install dependencies
npm run dev          # Dev server (Next.js + Socket.io) at localhost:3000
npm run build        # prisma generate + migrate deploy + next build
npm run test         # Run all tests once
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
npx prisma migrate dev  # Run database migrations (dev)
npx prisma db seed      # Seed the database
npm run admin -- help   # Admin CLI — scriptable driver for /admin (see docs/admin-cli.md)
```

### Admin CLI (for AI coding agents)
`scripts/admin-cli/` is a headless HTTP client that authenticates with its own nsec (or NIP-46 bunker) and speaks to the same `/api/*` endpoints the web UI uses. Any CLI coding agent (Claude Code, Codex, Cursor…) can drive it to manage servers, channels, roles, members, bans, bots, and emojis on any Obelisk instance. Role checks are enforced server-side via the same `requireRole()` guards — the CLI has no extra privileges. See `scripts/admin-cli/AGENT.md` for the agent-oriented cheat sheet and [docs/admin-cli.md](docs/admin-cli.md) for the full guide.

## Deployment (Self-Hosted Docker)

See [DEPLOY.md](DEPLOY.md) for full setup instructions.

### Infrastructure
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fabricio333/obelisk](https://github.com/Fabricio333/obelisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
