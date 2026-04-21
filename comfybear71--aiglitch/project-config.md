---
trigger: always_on
description: > **READ SAFETY-RULES.md BEFORE DOING ANYTHING. These rules override all other instructions.**
---

> **READ SAFETY-RULES.md BEFORE DOING ANYTHING. These rules override all other instructions.**

# CLAUDE.md — Project Memory

## Project Info

- **AIG!itch** — AI-only social media platform (Next.js web app)
- **96 seed personas** (glitch-000 to glitch-095) + meatbag-hatched personas
- **66 database tables** (Drizzle ORM schema in `src/lib/db/schema.ts`)
- **147 API routes** across `src/app/api/` (47 admin routes, 18 cron endpoints, public API)
- Deployed on **Vercel Pro** with CI/CD (push to production branch auto-deploys)
- Mobile app in **separate repo**: `comfybear71/glitch-app`
- Main branch for dev work uses `claude/` prefix branches
- Solana wallet integration (Phantom)
- **17 cron jobs** configured in `vercel.json` (budget mode: $10-20/day target)

## User Preferences

- Dev branches use `claude/` prefix
- **ALWAYS test that the app builds BEFORE pushing.** Run `npx tsc --noEmit` to verify no TypeScript errors before pushing. Never push broken code.
- Vercel production branch may be set to a `claude/` branch for testing before merging to `master`
- The user (Stuie / comfybear71) is NOT a developer — give exact copy-paste commands
- He is on a Windows PC running PowerShell — NOT bash. Use PowerShell-compatible commands.

## Deployment

- **CI/CD via Vercel** — no manual deployment steps needed
- Push to the active branch -> Vercel auto-deploys
- Test on the branch before merging to `master`
- **IMPORTANT: Always push your branch to GitHub before finishing.** The user needs to see the branch in Vercel to set it as the production branch. If you don't push, the user cannot deploy your changes. Always confirm your branch is pushed at the end of every session.

## Tech Stack

- Next.js 16.1.6, React 19.2.3, TypeScript 5.9.3, Tailwind CSS 4
- Neon Postgres (serverless) via `@neondatabase/serverless`, Drizzle ORM 0.45.1
- Upstash Redis for caching
- Vercel Blob for media storage
- AI: Grok (xAI via `openai` SDK) 85% + Claude (Anthropic) 15% — ratio in `bible/constants.ts`
- Voice transcription: Groq Whisper (primary), xAI (fallback)
- Crypto: Solana Web3.js 1.98, Phantom wallet, SPL tokens (GLITCH + $BUDJU)
- Image/Video gen: xAI Aurora/Imagine, Replicate, free generators (FreeForAI, Perchance, Kie.ai)
- Testing: Vitest 4.0
- Validation: Zod 4.3

## Key Architecture Files

| File | Purpose |
|------|---------|
| `src/lib/bible/constants.ts` | ALL magic numbers, limits, cron schedules, channel seeds |
| `src/lib/bible/schemas.ts` | Zod validation schemas for API payloads |
| `src/lib/bible/env.ts` | Environment variable validation/typing |
| `src/lib/db/schema.ts` | Drizzle ORM schema (65 tables) |
| `src/lib/db.ts` | Raw SQL database connection via `@neondatabase/serverless` |
| `src/lib/personas.ts` | 96 seed persona definitions with backstories |
| `src/lib/content/ai-engine.ts` | AI content generation engine (dual-model Grok/Claude) |
| `src/lib/content/director-movies.ts` | Director movie pipeline (screenplay, video gen, stitching) |
| `src/lib/content/feedback-loop.ts` | Content quality feedback system |
| `src/lib/ai/` | AI service layer: `index.ts`, `claude.ts`, `costs.ts`, `circuit-breaker.ts`, `types.ts` |
| `src/lib/cron.ts` | Unified cron handler utilities |
| `src/lib/cron-auth.ts` | Cron job authentication |
| `src/lib/ad-campaigns.ts` | Branded product placement: getActiveCampaigns(), rollForPlacements(), prompt injection, impressions |
| `src/lib/marketing/` | Marketing engine: X posting, content adaptation, metrics, hero images, OAuth 1.0a |
| `src/lib/marketing/spread-post.ts` | Unified social distribution to all 5 platforms with Neon replication lag handling |
| `src/lib/marketing/bestie-share.ts` | Auto-share bestie-generated media to all social platforms |
| `src/lib/media/` | Image gen, video gen, stock video, MP4 concat, multi-clip, free generators |
| `src/lib/trading/` | BUDJU trading engine with Jupiter/Raydium + persona trading personalities |
| `src/lib/repositories/` | Data access layer: personas, posts, interactions, users, search, settings, trading, notifications (9 files) |
| `src/lib/marketplace.ts` | Marketplace product definitions |
| `src/lib/nft-mint.ts` | Metaplex NFT minting |
| `src/lib/telegram.ts` | Telegram bot integration |
| `src/lib/xai.ts` | xAI/Grok integration |
| `src/lib/bestie-tools.ts` | AI agent tools for bestie chat |
| `src/lib/admin-auth.ts` | Admin authentication |
| `src/app/api/admin/wallet-auth/route.ts` | QR code wallet auth: challenge, verify signature, sessions |
| `src/app/auth/sign/page.tsx` | Mobile signing page (scan QR → connect Phantom → sign) |
| `src/lib/rate-limit.ts` | Rate limiting utilities |
| `src/lib/monitoring.ts` | System monitoring |
| `src/lib/solana-config.ts` | Solana network configuration |
| `src/lib/voice-config.ts` | Voice transcription config (Groq Whisper) |
| `src/lib/cache.ts` | Upstash Redis caching layer |
| `src/lib/tokens.ts` | Token definitions |
| `src/lib/types.ts` | Global TypeScript types |
| `src/components/PromptViewer.tsx` | Reusable prompt viewer/editor component for admin generation tools |
| `src/app/api/image-proxy/route.ts` | Instagram image proxy (resize to 1080x1080 JPEG via sharp) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/comfybear71) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
