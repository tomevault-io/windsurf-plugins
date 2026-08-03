---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **marketing, presentation, and download website** for **Netfox**, a native macOS app that monitors your home network — devices, history, and alerts.

**This is NOT a macOS application.** This is a Next.js web project deployed on Vercel.

- **Live URL** (TBD): https://netfox-website.vercel.app/
- **App repository** (private, Swift/SwiftUI): https://github.com/gfazioli/Netfox
- **Website repository** (this): https://github.com/gfazioli/netfox-website

The website serves as:
1. **Landing page** — hero section, feature showcase, download CTA
2. **Documentation** — user guides, getting started, FAQ
3. **Release notes** — pulled automatically from GitHub Releases API
4. **Download hub** — links to GitHub Releases for the macOS binary
5. **Sparkle appcast host** — `public/appcast.xml` is the EdDSA-signed update feed the app polls

## Tech Stack

- **Framework**: Next.js 16 + Nextra 4 (docs/MDX)
- **UI Library**: Mantine 9
- **Animations**: @gfazioli/mantine-scene, @gfazioli/mantine-text-animate, @gfazioli/mantine-marquee
- **Icons**: @tabler/icons-react
- **Analytics**: @vercel/analytics
- **Hosting**: Vercel
- **Package Manager**: Yarn 4 (Berry) — do not use npm or pnpm

## Commands

| Command | Purpose |
|---------|---------|
| `yarn dev` | Start Next.js dev server |
| `yarn build` | Production build (Next.js + pagefind search index) |
| `yarn test` | Full suite: typegen, oxfmt, lint, typecheck, jest |
| `yarn jest` | Run Jest tests only |
| `yarn typecheck` | TypeScript type checking (`tsc --noEmit`) |
| `yarn lint` | oxlint + Stylelint |
| `yarn format:write` | Auto-format all TS/TSX/CSS files (oxfmt) |
| `yarn storybook` | Storybook dev server on port 6006 |
| `yarn analyze` | Bundle analysis with `@next/bundle-analyzer` |

> **If `yarn <cmd>` fails with `command not found: oxfmt` / `next`** the Yarn PATH shim isn't wired on this machine — run the binary directly instead: `./node_modules/.bin/oxfmt`, `./node_modules/.bin/next dev`, `./node_modules/.bin/next build`. `yarn test` / `yarn jest` route through the npm-run shim and work regardless.

## Architecture

### Routing & Content

- **App Router** (`app/`): Next.js 16 app router with Nextra integration
- **Docs content** (`content/`): MDX files rendered via Nextra at `/docs/[[...mdxPath]]`
- Nextra is configured with `contentDirBasePath: '/docs'` — all MDX content is served under `/docs`
- `content/_meta.ts` controls sidebar navigation order and labels

### Layout & Theme Integration

- `app/layout.tsx` wraps the entire app in both `MantineProvider` and Nextra's `Layout`
- Dark mode sync between Mantine and Nextra is handled by `MantineNextraThemeObserver`
- Mantine theme overrides go in `theme.ts` (client-side `createTheme`)
- Global site configuration (metadata, GitHub API, search, Nextra layout) lives in `config/index.ts`
- Primary color: orange (matching Netfox app icon)
- Custom color palette: `netfox` (orange/foxy shades)

### Key Components (`components/`)

- `MantineNavBar` — top navigation with Netfox logo + GitHub link
- `MantineFooter` — 4-column footer with highlights, resources, ecosystem links
- `Welcome` — hero section with animated title, features grid, download CTA
- `ColorSchemeControl` / `ColorSchemeToggle` — dark mode toggle
- `ReleaseNotes` — fetches GitHub releases via `/api/github-releases`
- `ProblemSection` / `SolutionSection` / `BuiltForMacSection` — marketing sections used by `Welcome`
- `FAQ` — accordion-style FAQ, content driven by an array prop

### API Routes (`app/api/`)

- `version/` — returns current package version
- `github-releases/` — proxies GitHub Releases API for Netfox (configured in `config/index.ts`). Uses `GITHUB_TOKEN` env var when set to raise the rate limit from 60/hr to 5000/hr.
- `search/` — pagefind-based full-text search endpoint

### Environment variables

- `GITHUB_TOKEN` (optional, recommended on Vercel) — fine-grained or classic token with `public_repo` read scope. Used by:
  - The `/api/github-releases` proxy (runtime).
  - The `content/release-notes.mdx` TOC metadata, which fetches at build time so Vercel needs the var available during deploys.
  Without the token the app still works but may hit 60 req/hr GitHub rate limit on shared IPs.

### CSS Import Order

In `app/layout.tsx`, CSS imports must follow this order:
1. `@mantine/core/styles.css`
2. Mantine extension styles (marquee, text-animate, scene)
3. Global styles

## Content Guidelines

- All website content is in **English**
- The app is described as: "A native network monitor for macOS"
- Key selling points: device discovery (Bonjour + ARP + active probing), per-device history timeline, alerts for new devices, native macOS UI, no cloud account required, no telemetry
- Target audience: home network users who want to know who's connected, when, and from where — at a glance
- Download links point to GitHub Releases on the **website** repo (the app repo is private): `https://github.com/gfazioli/netfox-website/releases/latest`

### No infrastructure leaks in user-facing copy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gfazioli/netfox-website](https://github.com/gfazioli/netfox-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
