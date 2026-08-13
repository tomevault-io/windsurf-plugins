---
trigger: always_on
description: **This is the LIVE PRODUCTION shop at shop.govcongiants.com**
---

# GovCon Shop - Claude Project Context

## Quick Identifier
**This is the LIVE PRODUCTION shop at shop.govcongiants.com**

When user says: "live shop", "production", "shop.govcongiants", "the real site"
→ This is the project. **Handle with care - this is live!**

---

## Workflow Orchestration

### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimize Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---

## IMPORTANT: No Framer
**This project does NOT use Framer.** Do not use any Framer MCP tools (mcp__framer-mcp__*) for this project. This is a pure Next.js/React codebase.

---

## Project Overview
**Name:** GovCon Shop (Production)
**Purpose:** Live production shop for GovCon Giants tools
**Live URL:** shop.govcongiants.com
**Framework:** Next.js 16 with Turbopack
**Database:** Supabase
**Payments:** Stripe

## Project Location
```
/Users/ericcoffie/govcon-shop
```

---

## Related Projects

| Project | Location | Purpose |
|---------|----------|---------|
| **GovCon Funnels** | `/Users/ericcoffie/govcon-funnels` | Marketing site (govcongiants.org) |
| **Market Assassin** | `/Users/ericcoffie/Market Assasin/market-assassin` | Dev/staging environment |
| **GovCon Shop** | This project | **LIVE production shop** |

---

## Tech Stack
- **Frontend:** Next.js 16, React 19, TypeScript, Tailwind CSS
- **Backend:** Next.js API Routes
- **Database:** Supabase (PostgreSQL)
- **Auth:** Supabase Auth + Email-based access (no license keys required)
- **Payments:** Stripe (webhooks + payment links)
- **Email:** SMTP (hello@govconedu.com)
- **Access Gating:** Vercel KV (fast) + Supabase user_profiles (source of truth)

---

## Products & Pricing

### Premium Tools
| Product | Price |
|---------|-------|
| Market Assassin Standard | $297 |
| Market Assassin Premium | $497 |
| Content Reaper | $197 |
| Content Reaper Full Fix | $397 |
| Contractor Database | $497 |
| Recompete Tracker | $397 |
| Opportunity Hunter Pro | $49 |

### Bundles
| Bundle | Price | Savings |
|--------|-------|---------|
| Starter | $697 | $246 |
| Pro Giant | $997 | $391 |
| Ultimate | $1,497 | $291 |

---

## Key Pages

### Tools (Premium)
| Route | Tool |
|-------|------|
| `/market-assassin` | Federal Market Assassin |
| `/content-generator` | GovCon Content Reaper |
| `/contractor-database` | Federal Contractor Database |
| `/expiring-contracts` | Recompete Tracker |
| `/opportunity-hunter` | Opportunity Hunter |
| `/planner` | Action Planner Dashboard |

### Free Resources
| Route | Resource |
|-------|----------|
| `/free-resources` | All free resources hub |
| `/sblo-directory` | SBLO Contact List PDF |
| `/december-spend` | December Spend Forecast |
| `/ai-prompts` | 75+ AI Prompts PDF |
| `/action-plan-2026` | 2026 Action Plan |
| `/guides-templates` | Guides & Templates |
| `/tribal-list` | Tribal Contractor List |
| `/expiring-contracts-csv` | Free Contracts CSV |

### Blog / Resource Center
| Route | Purpose |
|-------|---------|
| `/resources.html` | GovCon Resource Center — 15 articles with topic filtering, full-page reader, tools sidebar |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecoffie/govcon-shop](https://github.com/ecoffie/govcon-shop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
