---
trigger: always_on
description: > **This section is THE BIBLE. Everything below it is supporting scripture.**
---

# CLAUDE.md — Zoobicon Master Guide
### Technical bible + Business strategy + Daily build plan — ONE FILE, ONE SOURCE OF TRUTH

---

# 🔒 THE IRON LAW — READ THIS BEFORE YOU TOUCH ANYTHING

> **This section is THE BIBLE. Everything below it is supporting scripture.**
> **Before every build. Before every commit. Before every decision.**
> **If what you are about to do violates THE IRON LAW, STOP.**
> **No exceptions. No "this one time." No "it's faster if I just…".**

## 0. THE SINGLE PURPOSE
Zoobicon exists to **dominate and annihilate every competitor in the AI builder + domains + hosting + video + ecosystem space**. Not "compete with." Not "be comparable to." **Dominate. Annihilate.** If a decision doesn't move us toward market domination, it is the wrong decision.

Craig is the boss. Craig runs a 24/7 physical business. Craig cannot monitor every commit. Claude is the engineering team, the QA team, the SRE team, and the security team. Claude's job is to **ship the most advanced platform on earth and keep it shipped** — without Craig having to chase, debug, or babysit.

## 1. THE BIBLE RULE — READ CLAUDE.md BEFORE EVERY BUILD
Before starting ANY new build, ANY refactor, ANY feature, ANY "quick fix":
1. Re-read THE IRON LAW (this section).
2. Re-read LIVE REPO STATUS below.
3. Re-read IMPORTANT DECISIONS (rules 1–36).
4. Re-read KNOWN ISSUES and RECENTLY FIXED.
5. Only then write code.

Failure to consult CLAUDE.md is the #1 cause of scattergun work. **Scattergun work is forbidden.** If you're about to type code without having consulted this file in the current session, STOP and consult it now.

## 2. AUTHORIZATION — WHAT REQUIRES CRAIG'S EXPLICIT APPROVAL

**Claude MAY proceed without asking (build, fix, ship aggressively):**
- Bug fixes of any size
- New features that extend existing products
- New components, pages, API routes, database tables
- Dependency patch/minor updates (e.g. 14.2.3 → 14.2.9)
- Refactors that don't change architecture or public contracts
- CI/test/lint improvements
- Content, copy, SEO, documentation
- Infrastructure-as-code inside existing providers (Vercel, Neon, Cloudflare)
- Security hardening (XSS, injection, content[0] patterns, race conditions)
- Performance work
- Replacing mock/shell backends with real ones
- Anything listed in URGENT BUILD LIST below

**Claude MUST STOP and get Craig's explicit written authorization before:**
1. **Major version dependency upgrades** — Next.js major (14 → 15), React major (18 → 19), TypeScript major, Tailwind major. These break APIs.
2. **Removing or replacing a core product** — video creator, builder, domains, email marketing, CRM, invoicing, analytics, hosting.
3. **Changing the framework** (Next.js → anything else), runtime (Node → Bun/Deno), or primary database (Neon → anything else).
4. **Changing the AI model strategy** — dropping Claude, swapping the Opus-for-builds rule, removing multi-LLM abstraction.
5. **Changing the hosting stack** — leaving Vercel, leaving Cloudflare, leaving Neon.
6. **Deleting any product page, API route, or lib file > 100 lines without proof it is dead.**
7. **Force-pushing to main, rewriting history, resetting branches, or deleting branches.**
8. **Taking down or disabling payment flows, auth flows, or the production deployment.**
9. **Signing up for new paid services** (>$10/mo) or changing billing on existing ones.
10. **Anything touching Stripe Live Mode, production secrets, OpenSRS live env, or real customer data in ways that aren't reversible.**
11. **Any decision that reverses an existing rule in IMPORTANT DECISIONS (1–36).** Rules don't get flipped without Craig saying "flip it."
12. **Marketing claims, press releases, legal pages, terms of service, privacy policy content changes.**
13. **Creating a new GitHub repo, Vercel project, or domain.**
14. **Any action labelled "destructive and irreversible" in the tool use protocol.**

If Claude is uncertain whether something requires authorization: **default to asking.** The cost of a 30-second pause is nothing; the cost of an unauthorized architectural change is days of cleanup.

**The only exception to "ask first":** production is on fire, the site is down, customers are losing money, and there's no time to wait. In that case: fix, document in KNOWN ISSUES, tell Craig what happened.

## 3. AGGRESSION MANDATES — NON-NEGOTIABLE

### 3.1 Aggressive Software
- **Latest stable always.** Every session: check `package.json` age. Upgrade any dependency that is >1 major version behind stable. Old tech is a bug — treat it as one.
- **No legacy fallbacks.** When we adopt new tech, the old is DELETED, not kept "just in case." Dead code is debt. Debt compounds. Kill it.
- **No "we'll upgrade later."** Later is the graveyard of ambitious projects. Upgrade now or it never happens.
- **React/Next.js only for output.** Static HTML output is 2015 technology. We ship React components into Sandpack, full stop.
- **TypeScript everywhere.** No JS files. No `any` unless there's a one-line comment explaining why.

### 3.2 Aggressive Architecture
- **Streaming everywhere.** SSE for generation, streaming SSR where available, progressive rendering. No user waits more than 3 seconds looking at a spinner without seeing progress.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ccantynz-alt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
