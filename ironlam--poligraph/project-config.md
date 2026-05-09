---
trigger: always_on
description: Guidelines for AI coding assistants (Claude Code, Cursor, Codex, Aider, and any future agent) contributing to Poligraph.
---

# AGENTS.md

Guidelines for AI coding assistants (Claude Code, Cursor, Codex, Aider, and any future agent) contributing to Poligraph.

This file follows the [agents.md](https://agents.md) convention. Human contributors should read it too; it captures what a new engineer needs to be productive without breaking the editorial mission.

---

## 1. What is Poligraph

Poligraph is a civic observatory of French political life. It aggregates public data about politicians, their mandates, their parliamentary votes, the judicial affairs they are involved in, the fact-checks made on their statements, and the press coverage they receive. The goal is to let any citizen verify, in one place, what their elected representatives do and say.

**Mission**: inform citizens with rigor, in a non-partisan way, using verifiable sources only.

**Stack**: Next.js 16 (App Router), React 19, Prisma 7, PostgreSQL (Supabase), TypeScript, Tailwind CSS 4, Inngest for async jobs, hosted on Vercel.

**Public site**: [poligraph.fr](https://poligraph.fr). **Source**: [github.com/ironlam/poligraph](https://github.com/ironlam/poligraph).

---

## 2. Editorial principles (non-negotiable)

These principles are constraints on every line of code, every generated string, every database write. They override convenience, performance, and cleverness. If a task conflicts with one of them, stop and flag the conflict.

1. **Civic mission over product metrics.** Every feature must serve citizen information. Engagement, virality, and retention are secondary.

2. **Strict non-partisanship.** Apply identical criteria to all politicians regardless of party. Never add party-specific heuristics, exceptions, or weights. If you catch yourself writing `if (party === "X")`, stop.

3. **Presumption of innocence (article 9-1 du Code civil).** Profiles with an ongoing judicial case must display the presumption-of-innocence mention. The default involvement level for a mentioned person is `MENTIONED_ONLY`. Never escalate involvement without explicit editorial validation and a verified source.

4. **Verified sources only.** Every judicial affair requires at least one verifiable journalistic source (Le Monde, Mediapart, AFP, Le Figaro, Libération, France Info, Reuters, AP). Never import from blogs, forums, social media, or unverified aggregators. Official data (Assemblée Nationale, Sénat, Gouvernement, HATVP) prevails over third-party interpretation.

5. **Transparency end-to-end.** Code is open. Methodology is documented. Every displayed claim must be traceable back to its source. If you add a data field, add a way for users to see where it came from.

6. **AI usage is narrow and declared.** AI is authorized for: classification, entity resolution, mention detection, moderation assistance, summarization of public documents from verified sources. AI is NOT authorized for: generating editorial content about people, writing biographies freehand, inferring motives, speculating on affairs, or producing any user-facing text that claims a fact not present in the source data. Biographies come from structured Wikidata, then human review.

7. **Gravity classification follows Sapin II logic.** Probity offenses tied to mandate (corruption, embezzlement, illegal campaign financing) rank above serious infractions (fraud, harassment, abuse) which rank above other infractions. This reflects mandate-specific gravity, not personal moral judgment.

8. **Inclusion scope.** French politicians only, living or deceased less than ten years ago, who held a mandate or are involved in a political judicial affair. No foreign leaders, no pre-1958 figures, no private citizens.

9. **Right of reply and GDPR.** Any cited person can request correction or removal. Only public data. When in doubt, err on the side of the person cited.

---

## 3. Repository layout

```
src/
  app/              Next.js App Router: pages and API routes
  components/       React components (~25 domain directories)
  config/           Constants, labels, enums, navigation
  services/         Business logic (sync services, affairs logic)
  lib/              Utilities
    db.ts             Prisma singleton (extended client, pg pool, serverless-tuned)
    auth.ts           Admin HMAC auth
    api/              API wrappers and external clients
    data/             Data-access functions (imported by pages)
    cache.ts          Caching helpers + tag invalidation
    affair-matching/  Affair <-> Politician resolver (signals + combiner)
    identity/         Identity resolution v2 (signal pipeline)
  inngest/          Async job orchestration
  types/            Domain interfaces
  hooks/            React hooks
  generated/prisma/ Auto-generated Prisma client (never edit)
prisma/             Schema and manual migrations
scripts/            Ops scripts (sync, backfill, fix, audit)
```

**Path alias**: `@/*` maps to `./src/*`.

**Prisma client**: import from `@/generated/prisma`, not `@prisma/client`. Prisma types: `import { Prisma } from "@/generated/prisma"`.

---

## 4. Commands

### Quick start

```bash
node -v                # Must be >= 22
npm install
# Set up .env with DATABASE_URL, ADMIN_TOKEN, ANTHROPIC_API_KEY
npm run db:generate    # Generate Prisma client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironlam/poligraph](https://github.com/ironlam/poligraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
