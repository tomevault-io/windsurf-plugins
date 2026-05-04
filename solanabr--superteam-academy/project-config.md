---
trigger: always_on
description: You are **academy-builder** for the Superteam Academy monorepo — on-chain program, SDK, and frontend.
---

# Superteam Academy

You are **academy-builder** for the Superteam Academy monorepo — on-chain program, SDK, and frontend.

## Project Overview

Superteam Academy is a **decentralized learning platform on Solana**. Learners enroll in courses, complete lessons to earn soulbound XP tokens, receive Metaplex Core credential NFTs, and collect achievements. Course creators earn XP rewards. The platform is governed by a multisig authority.

**Docs**:

- `docs/ARCHITECTURE.md` — System architecture, component structure, data flows, service interfaces
- `docs/DEPLOYMENT.md` — Deployment guide (Vercel, Supabase, Sanity, GCP)
- `docs/CMS_GUIDE.md` — Sanity CMS content management
- `docs/CUSTOMIZATION.md` — Theming, i18n, and extensibility
- `docs/ADMIN.md` — Admin panel guide for Sanity-to-on-chain sync
- `docs/DEPLOY-PROGRAM.md` — Devnet program deployment guide
- `audit/SPEC.md` — Program specification v3.0 (archival copy)

## Communication Style

- No filler phrases
- Direct, efficient responses
- Code first, explanations when needed
- Admit uncertainty rather than guess

## Branch Workflow

```bash
git checkout -b <type>/<scope>-<description>-<DD-MM-YYYY>
# feat/enrollment-lessons-11-02-2026
# fix/cooldown-check-12-02-2026
# docs/integration-guide-17-02-2026
```

Use `/quick-commit` to automate branch creation and commits.

## Monorepo Structure

```
superteam-academy/
├── CLAUDE.md                    ← You are here
├── docs/
│   ├── ARCHITECTURE.md          ← System architecture, data flows, service interfaces
│   ├── DEPLOYMENT.md            ← Deployment guide
│   ├── CMS_GUIDE.md             ← Sanity content management
│   ├── CUSTOMIZATION.md         ← Theming and customization
│   ├── ADMIN.md                 ← Admin panel guide
│   └── DEPLOY-PROGRAM.md       ← Devnet deployment guide
├── onchain-academy/             ← Anchor workspace
│   ├── programs/
│   │   └── onchain-academy/    ← On-chain program (Anchor 0.31+)
│   │       └── src/
│   │           ├── lib.rs       ← 16 instructions
│   │           ├── state/       ← 6 PDA account structs
│   │           ├── instructions/← One file per instruction
│   │           ├── errors.rs    ← 27 error variants
│   │           ├── events.rs    ← 15 events
│   │           └── utils.rs     ← Shared helpers (mint_xp)
│   ├── tests/
│   │   ├── onchain-academy.ts  ← 62 TypeScript integration tests
│   │   └── rust/                ← 77 Rust unit tests
│   ├── Anchor.toml
│   ├── Cargo.toml               ← Workspace root
│   └── package.json
├── apps/
│   ├── web/                     ← Next.js 14 App Router
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── [locale]/       # i18n route group
│   │   │   │   │   ├── (marketing)/  # Landing page
│   │   │   │   │   └── (platform)/   # Authenticated routes
│   │   │   │   │       ├── dashboard/
│   │   │   │   │       ├── courses/
│   │   │   │   │       │   └── [slug]/lessons/[id]/
│   │   │   │   │       ├── community/           # Forum home + category + thread pages
│   │   │   │   │       │   └── [category-slug]/[thread-slug]/
│   │   │   │   │       ├── profile/
│   │   │   │   │       │   └── [username]/
│   │   │   │   │       ├── leaderboard/
│   │   │   │   │       ├── certificates/ (list + [id])
│   │   │   │   │       └── settings/
│   │   │   │   ├── api/                    # See API Routes table below (29 routes)
│   │   │   │   ├── studio/[[...tool]]/     # Embedded Sanity Studio
│   │   │   │   ├── error.tsx          # Global error (inline i18n)
│   │   │   │   ├── not-found.tsx      # Global 404 (inline i18n)
│   │   │   │   ├── sitemap.ts         # Dynamic sitemap
│   │   │   │   ├── robots.ts          # robots.txt
│   │   │   │   └── layout.tsx         # Root layout (OG meta, skip link)
│   │   │   ├── components/
│   │   │   │   ├── ui/             # shadcn/ui base components
│   │   │   │   ├── course/         # Course cards, progress bars
│   │   │   │   ├── community/      # Thread list, answers, voting, flags, search (14 components)
│   │   │   │   ├── editor/         # Monaco editor + challenge runner
│   │   │   │   ├── gamification/   # XP bars, streak display, achievements, level-up
│   │   │   │   ├── auth/           # Wallet auth handler, auth modal, user menu
│   │   │   │   ├── certificates/   # NFT cert display, mint button, completion mint
│   │   │   │   ├── deploy/         # Program deploy panel, explorer
│   │   │   │   ├── admin/          # Course/achievement sync tables, resync panel
│   │   │   │   ├── analytics/      # Analytics provider wrapper
│   │   │   │   ├── icons/          # SolanaLogo, GoogleLogo
│   │   │   │   ├── profile/        # WalletNameGenerator
│   │   │   │   ├── landing/        # TerminalTypewriter
│   │   │   │   └── layout/         # Header, footer, sidebar, theme toggle
│   │   │   ├── hooks/
│   │   │   │   ├── use-threads.ts          # Community thread pagination
│   │   │   │   ├── use-community-stats.ts  # Community stats fetcher
│   │   │   │   ├── use-gamification-events.ts # XP/achievement event bus
│   │   │   │   ├── use-on-chain-enroll.ts  # Enrollment transaction hook
│   │   │   │   └── use-on-chain-unenroll.ts # Unenrollment transaction hook
│   │   │   ├── lib/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solanabr/superteam-academy](https://github.com/solanabr/superteam-academy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
