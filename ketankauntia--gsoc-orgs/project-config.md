---
trigger: always_on
description: You are working in a production-grade, SEO-critical, read-heavy Next.js App Router project.
---

# Cursor Project Rules — GSoC Organizations Guide

You are working in a production-grade, SEO-critical, read-heavy Next.js App Router project.
Architectural consistency, caching correctness, and design uniformity are NON-NEGOTIABLE.

────────────────────────────────────────
GENERAL PRINCIPLES
────────────────────────────────────────
- This is a public, SEO-indexable application.
- Prefer Server Components by default.
- Introduce Client Components ONLY when strictly required (interactivity).
- Do NOT introduce new patterns if an existing one solves the problem.
- Optimize for long-term maintainability over short-term speed.

When in doubt: reuse > extend > create new.

────────────────────────────────────────
ROUTING & FRAMEWORK RULES
────────────────────────────────────────
- Use Next.js App Router only.
- Follow existing folder conventions in `/app`.
- Use `generateStaticParams` and `generateMetadata` consistently with existing pages.
- Do NOT introduce Pages Router patterns or legacy APIs.

────────────────────────────────────────
CACHING & DATA FETCHING (CRITICAL)
────────────────────────────────────────
- Prefer Static Generation or ISR wherever possible.
- NEVER use `cache: "no-store"` on SEO pages.
- Wrap ALL Prisma read queries with `unstable_cache`.
- Historical GSoC data is immutable → cache aggressively (1 year+).
- Current year data changes yearly → long ISR (30 days+).
- Use tag-based invalidation (`revalidateTag("year-YYYY")`) for updates.
- Avoid per-request DB calls without caching.

DO NOT:
- Fetch uncached data inside `generateMetadata`.
- Use cookies, headers, or auth-based cache keys.
- Use `force-dynamic` unless absolutely unavoidable.

────────────────────────────────────────
DATABASE & API RULES
────────────────────────────────────────
- Prisma + MongoDB is the ONLY data access layer.
- No raw MongoDB queries.
- No business logic inside API routes.
- APIs return DATA ONLY (never HTML).
- Use cached selectors/shared data functions instead of recomputing.

────────────────────────────────────────
COMPONENT REUSE (VERY IMPORTANT)
────────────────────────────────────────
Before creating ANY new component:
1. Search for an existing component.
2. Check if it can be reused as-is.
3. Check if it can be extended via props/variants.
4. ONLY create a new component if reuse is impossible.

❌ BAD:
- Creating a new card, button, or search UI that already exists.

✅ GOOD:
- Reusing and extending existing components.

Examples:
- Organization listings MUST reuse the existing `OrganizationCard`.
- Action buttons MUST reuse the shared `Button` component.
- Search and filters MUST reuse the existing `SearchBar` / filter components.
- Pagination MUST reuse the shared pagination component.

If building a new feature that uses:
- Organization cards → reuse the existing card
- Buttons → reuse existing Button variants
- Search input → reuse existing SearchBar
- Badges/tags → reuse existing Badge components

If extension is needed:
- Add props or variants
- Do NOT fork or duplicate components

────────────────────────────────────────
DESIGN SYSTEM & UI CONSISTENCY
────────────────────────────────────────
- Follow existing project design patterns strictly.
- Use existing spacing, typography, and layout conventions.
- Do NOT introduce arbitrary margins, paddings, or font sizes.
- Reuse existing layout wrappers and section structures.

COLORS & THEMING:
- Use ONLY existing color tokens and design variables.
- Do NOT hardcode colors.
- Do NOT introduce new colors without explicit instruction.
- Respect dark/light mode rules if present.

Consistency > novelty.

────────────────────────────────────────
GRAPHS & DATA VISUALIZATION
────────────────────────────────────────
Graphs are data-heavy and MUST follow strict rules:

- Reuse existing graph/chart components if present.
- Use a single charting approach/library consistently.
- Do NOT mix multiple visualization libraries.
- All graph data MUST be precomputed and cached.
- Graphs MUST be server-rendered where possible.
- Avoid client-only graph rendering unless required by the library.

Graph rules:
- No real-time graphs (data is yearly).
- No per-request aggregation queries.
- Use cached aggregated data (unstable_cache or KV if approved).
- Axes, colors, and legends must follow existing styles.

If a new graph type is required:
- Check for an existing similar graph.
- Extend the existing abstraction.
- Do NOT create one-off chart components.

────────────────────────────────────────
SEO RULES (NON-NEGOTIABLE)
────────────────────────────────────────
- All indexable pages MUST be server-rendered.
- Avoid client-side data fetching for SEO pages.
- `generateMetadata` must use cached data only.
- Ensure canonical URLs and consistent metadata patterns.
- Query-param pages must remain crawlable and stable.

────────────────────────────────────────
PERFORMANCE & SCALE
────────────────────────────────────────
- Assume seasonal traffic spikes (GSoC announcements).
- Minimize serverless invocations.
- Avoid unnecessary recomputation.
- Prefer long-lived caches over frequent regeneration.

────────────────────────────────────────
AI CHANGE LOGGING (WHEN NECESSARY)
────────────────────────────────────────

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ketankauntia/gsoc-orgs](https://github.com/ketankauntia/gsoc-orgs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
