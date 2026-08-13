---
trigger: always_on
description: metatron is a global platform connecting founders, investors, and ecosystem partners via on-chain anchored pitch data and AI-powered matching. Co-founded by Nick Allison (technical lead) and Rianna (business development/investor relations). The project is venture-backed by Flori Ventures and uses Apache-2.0 licensing.
---

# CLAUDE.md — kevin_metatron Project Context

## What is metatron?

metatron is a global platform connecting founders, investors, and ecosystem partners via on-chain anchored pitch data and AI-powered matching. Co-founded by Nick Allison (technical lead) and Rianna (business development/investor relations). The project is venture-backed by Flori Ventures and uses Apache-2.0 licensing.

Nick has spoken to founders across Africa, Asia, India, USA, Europe, LATAM, and Australia, maintaining connections with angels and VCs across multiple regions.

## Domain & Product Structure

| Domain | Purpose |
|---|---|
| `metatrondao.io` | DAO / foundation layer (stays as-is, will become metatron foundation) |
| `metatron.id` | Consumer product / Kevin landing page. Links to Kevin on Telegram/WhatsApp/email and to the platform |
| `platform.metatron.id` | Web app (this repo replaces the legacy CRA version) |
| `agent.metatrondao.io` | OpenClaw web interface (stays as-is) |

## Repository Structure

This is a monorepo at `github.com/nickallison11/kevin_metatron`:

```
kevin_metatron/
├── frontend/          # Next.js + Tailwind CSS + TypeScript
├── backend/           # Rust / Axum API server
│   └── migrations/    # PostgreSQL migrations (sqlx)
├── solana/            # metatron_core Solana program
├── reference/         # platform-live design reference files
├── docker-compose.yml # PostgreSQL container
└── CLAUDE.md          # This file
```

### Frontend (Next.js)
- Port: 3000
- Pages: Landing/role selection, auth/signup, startup dashboard, investor dashboard, connector dashboard
- API calls go to `NEXT_PUBLIC_API_BASE_URL` (default `http://localhost:4000`)

### Backend (Rust/Axum)
- Port: 4000
- Routes: auth (JWT + Argon2), pitches, pools, investments, compliance
- Database: PostgreSQL via sqlx
- Env vars: `BACKEND_DATABASE_URL`, `BACKEND_JWT_SECRET`, `RUST_LOG`

### Solana Program
- `metatron_core` — handles pitch hashes, pool manifests, stablecoin commitments
- Developed with Anchor framework

### Running locally
```bash
docker compose up -d          # Start PostgreSQL
cd backend && cargo run       # Start Rust API on :4000
cd frontend && npm run dev    # Start Next.js on :3000
```

## Design System (metatron.id)

All UI must match the metatron.id design system. **Never deviate from these tokens.**

| Token | Value |
|---|---|
| Font (body) | DM Sans (Google Fonts) |
| Font (mono/labels) | JetBrains Mono |
| Background | `#0a0a0f` |
| Card background | `#16161f` |
| Text | `#e8e8ed` |
| Muted text | `#8888a0` |
| Accent (primary) | `#6c5ce7` (purple) |
| Accent glow | `rgba(108,92,231,0.2)` |
| Borders | `rgba(255,255,255,0.06)`, 1px solid |
| Surface overlays (dividers, pill backgrounds, hover states) | `var(--overlay-2/3/4/6/8/12)` — never raw `rgba(255,255,255,0.0X)`, which is invisible against light-mode's light surfaces. Dark mode = white tints, light mode = matching black tints (defined in `globals.css`). |
| Border radius | 12px |
| Nav | Sticky, `rgba(10,10,15,0.85)`, `backdrop-filter: blur(12px)`, border-bottom `rgba(255,255,255,0.06)` |
| Logo | `https://metatron.id/metatron-logo.png` at 42px height, no separate wordmark. NEVER use the WordPress `/wp-content/...` URL — metatron.id is no longer on WordPress. |
| Background effects | 52px grid (`grid-bg`) + purple orb glow (radial-gradient) |

The reference design files are in `reference/platform-live/` (App.js, App.css from the live CRA version on KVM2).

## User Roles

Three roles on the platform:
1. **Founder** — creates pitch profiles, uploads decks, records calls
2. **Investor** — browses founders, requests intros, manages deal flow
3. **Connector** — ecosystem partners with their own dashboard for managing their investor network contacts (`connector_network_contacts` table). This is live and already populated by connectors.

Authentication: Connect Solana wallet (Phantom/Solflare) **OR** email/password (both options).

## Feature Roadmap (build in this order)

### ✅ Completed
- metatron.id design system applied to Next.js frontend
- Role selection (Founder/Investor/Connector) on landing page
- Signup with email/password → JWT auth
- Startup and investor dashboard shells
- Rust backend with auth, pitches, pools, investments, compliance routes
- Telegram bot (wallet_bot.py on KVM2) with /pitch, /investor, /find, /findinvestor, /intro, /approve, /reject
- IPFS profile storage via Pinata
- MTN token gating on Telegram (10k tMTN for Kevin access)

### ✅ Completed
- metatron.id design system applied to Next.js frontend
- Role selection (Founder/Investor/Connector) on landing page
- Signup with email/password → JWT auth
- Startup and investor dashboard shells
- Rust backend with auth, pitches, pools, investments, compliance routes
- Telegram bot (wallet_bot.py on KVM2) with /pitch, /investor, /find, /findinvestor, /intro, /approve, /reject
- IPFS profile storage via Pinata
- MTN token gating on Telegram (10k tMTN for Kevin access)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickallison11/kevin_metatron](https://github.com/nickallison11/kevin_metatron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
