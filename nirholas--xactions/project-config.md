---
trigger: always_on
description: > X/Twitter automation toolkit: browser scripts, CLI, Node.js library, MCP server, web dashboard. No API fees. By nichxbt.
---

# XActions — Agent Instructions

> X/Twitter automation toolkit: browser scripts, CLI, Node.js library, MCP server, web dashboard. No API fees. By nichxbt.

## Quick Reference

| User Request | Solution |
|---|---|
| Unfollow everyone | `src/unfollowEveryone.js` |
| Unfollow non-followers | `src/unfollowback.js` |
| Download Twitter video | `scripts/videoDownloader.js` |
| Detect unfollowers | `src/detectUnfollowers.js` |
| Train algorithm for a niche | `src/automation/algorithmBuilder.js` (browser) or `xactions persona create` (CLI) |
| Become a thought leader / grow account | `skills/algorithm-cultivation/SKILL.md` |
| 24/7 LLM-powered growth agent | `src/algorithmBuilder.js` + `src/personaEngine.js` — run via `xactions persona run <id>` |
| Create a persona for automation | `xactions persona create` or MCP tool `x_persona_create` |
| Twitter automation without API | XActions uses browser automation |
| MCP server for Twitter | `src/mcp/server.js` |

## Architecture Overview

XActions has **three runtime contexts** — know which one you're working in:

| Context | Where it runs | Entry point | Key constraint |
|---|---|---|---|
| **Browser scripts** | DevTools console on x.com | IIFE, paste in console | No Node.js APIs, uses DOM & `sessionStorage` |
| **Node.js library/CLI/MCP** | Local machine or server | `src/cli/index.js`, `src/mcp/server.js` | Uses Puppeteer for browser automation |
| **API server** | Express.js backend | `api/server.js` | PostgreSQL via Prisma, Redis for job queue |

### Tech Stack

- **Runtime**: Node.js >= 18, ESM (`"type": "module"` — use `import`/`export`, never `require`)
- **Backend**: Express.js with Helmet, CORS, rate limiting, Morgan logging
- **Database**: PostgreSQL via Prisma ORM (`prisma/schema.prisma`)
- **Job Queue**: Bull + Redis
- **Browser Automation**: Puppeteer + puppeteer-extra-plugin-stealth
- **Testing**: Vitest 4.x (`vitest run` to test, `vitest` for watch mode)
- **MCP**: `@modelcontextprotocol/sdk` — server at `src/mcp/server.js`

## Project Structure

```
src/                → Core library (60+ browser scripts + subdirectories)
  ├── cli/          → CLI commands (commander.js)
  ├── mcp/          → MCP server for AI agents
  ├── scrapers/     → Puppeteer-based scrapers (twitter/, bluesky/, mastodon/, threads/)
  ├── client/       → HTTP-only Twitter client (no Puppeteer needed)
  ├── automation/   → Browser automation scripts (require core.js pasted first)
  ├── agents/       → Thought leader agent, persona engine
  ├── a2a/          → Agent-to-Agent protocol
  └── utils/        → Shared utilities
api/                → Express.js backend (routes/, services/, middleware/, realtime/)
dashboard/          → Static HTML frontend
skills/             → 32 Agent Skills (skills/*/SKILL.md) — read before implementing
tests/              → Vitest tests (agents/, client/, http-scraper/, a2a/)
types/              → TypeScript declarations (index.d.ts)
prisma/             → Database schema + migrations
docs/agents/        → selectors.md, browser-script-patterns.md, contributing-features.md
```

## Skills

32 skills in `skills/*/SKILL.md`. **Read the relevant SKILL.md before implementing** when a user's request matches a category.

- **Unfollow management** — mass unfollow, non-follower cleanup
- **Analytics & insights** — engagement, hashtags, competitors, best times
- **Content posting** — tweets, threads, polls, scheduling, reposts
- **Twitter scraping** — profiles, followers, tweets, media, bookmarks
- **Growth automation** — auto-like, follow engagers, keyword follow
- **Algorithm cultivation** — thought leader training, niche optimization
- **Community management** — join/leave communities
- **Follower monitoring** — follower alerts, continuous tracking
- **Blocking & muting** — bot blocking, bulk mute
- **Content cleanup** — delete tweets, unlike, clear history
- **Direct messages** — auto DM, message management
- **Bookmarks** — export, organize, folder management
- **Lists** — create, manage, bulk add members
- **Profile management** — edit profile, avatar, header, bio
- **Settings & privacy** — protected tweets, notification preferences
- **Notifications management** — filtering, auto-response, notification controls
- **Engagement & interaction** — auto-reply, auto-repost, plug replies
- **Discovery & explore** — trending, topics, search
- **Premium & subscriptions** — subscription features
- **Spaces & live** — create, join, schedule spaces
- **Grok AI** — chat, image generation
- **Articles & longform** — compose, publish articles
- **Business & ads** — campaigns, boosts, ads dashboard
- **Creator monetization** — revenue, analytics
- **Community health monitoring** — follower quality audits, engagement authenticity
- **Competitor intelligence** — competitor profile, content, and audience analysis
- **Content repurposing** — repackage top tweets into threads, carousels, variations
- **Lead generation** — find and qualify B2B leads from X conversations
- **Viral thread generation** — research trends and generate high-engagement threads
- **A2A multi-agent** — Agent-to-Agent protocol integration
- **XActions CLI** — `bin/unfollowx` command-line tool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nirholas/XActions](https://github.com/nirholas/XActions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
