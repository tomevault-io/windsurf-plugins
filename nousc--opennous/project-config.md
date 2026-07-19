---
trigger: always_on
description: This file helps AI coding assistants understand the architecture before making changes.
---

# Nous — AI Agent Guide

This file helps AI coding assistants understand the architecture before making changes.

## What this project is

Nous is the context graph for GTM agents. It resolves every person, conversation, and touchpoint across the GTM tool stack (Apollo, HubSpot, Smartlead, Gmail, LinkedIn) into one account record, ingests signals from email, LinkedIn, and calendar, and exposes that context via an MCP server and REST API so agents always have the full account in a single call.

## Monorepo structure

```
apps/
  api/       — Node.js/Express REST API (the /v2 Context API is the public surface)
  mcp/       — MCP server (@opennous/mcp, 24 tools — stdio bin + hosted HTTP variant)
  frontend/  — Vite + React + shadcn/ui (People, Companies, GTM Context, Lead Lists pages)
  worker/    — Background workers (CalendarPoller, signal ingestion, webhooks)
packages/
  core/      — Shared DB logic, Supabase client, the entity/claim/observation substrate
```

## Layer rules

- `packages/core` is the single source of truth for DB queries and substrate types. All apps import from here — never duplicate DB logic in an app.
- `apps/api` and `apps/mcp` are thin shells: they handle transport (HTTP / MCP protocol) and delegate all business logic to `packages/core`.
- `apps/worker` imports from `packages/core` for DB writes but owns its own polling/scheduling logic.
- `apps/frontend` never calls the DB directly — only calls `apps/api` endpoints.

## Key concepts

**The substrate** — everything reduces to three primitives. Agents never overwrite; they observe, and Nous derives:
- `entities` — canonical, durable anchors (a person or a company). The same person-entity survives a job change or a new email.
- `observations` — append-only log of what happened or was learned (an interaction, or a stated fact). The single write verb.
- `claims` — the current facts Nous *derives* from observations, each carrying a confidence and a freshness.

**Scopes** — a claim is attached to a contact entity, a company entity, or the workspace entity:
- contact — facts about one person (communication style, authority level)
- company — org-level facts shared across contacts at that company (budget cycles, deal history)
- workspace — the user's own GTM profile (ICP, market, pricing, positioning) — see the GTM Context page

**Signals** — email, LinkedIn messages, calendar meetings, calls, plus public signals (job postings, funding, tech-stack changes via webhooks) all land as observations against the resolved entity.

**Focus resolution** — an agent passes whatever it has. A hard identifier (entity UUID, email, LinkedIn URL, or domain) resolves to exactly one entity. A bare name is searched: zero hits → not found, one hit → resolved, several → the caller gets candidates to disambiguate (never auto-merge on name alone). Logic lives in `resolveFocus` in `packages/core/src/db/entities.ts`. Inbound signal matching adds a corroboration step that attaches a known contact's new email only when domain/company corroborates — see `apps/worker/src/utils/identityMatch.mjs`.

## Database

Supabase (PostgreSQL). Key tables (the v2 substrate):
- `entities` — canonical person/company anchors
- `entity_identifiers` — the emails, domains, LinkedIn URLs and external ids that resolve to an entity
- `observations` — append-only log of events and stated facts
- `claims` — the current derived facts per entity (with confidence + freshness)
- `predictions` — derived forecasts, including the latest `icp_fit` score per entity
- `relationships` — entity-to-entity edges (e.g. `works_at`, buying-group ties)
- `contacts` / `companies` — **views** over the v2 substrate (one flat profile row per entity, assembled from `entity_identifiers` + `claims` + `predictions`), with `INSTEAD OF` triggers so legacy writes still work. They are no longer real tables. Changing a column means editing the view in `supabase/schema.sql` (canonical) plus a dated migration, keeping the column list/order identical so the triggers keep matching.

**People vs leads & the pipeline.** The `contacts` view is also the People page, and it does NOT show every person — only ones you've actually engaged: an inbound reply (received LinkedIn message, email reply/received), a meeting/deal, a CRM record, a manual add, or `pipeline_stage` past the top of funnel. Cold/scraped leads and people you only messaged outbound stay out (they live as leads, queryable by the agent, not on People). Pipeline stages, low→high: `identified → aware → connected → interested → evaluating → client`. `connected` = an accepted LinkedIn connection with no conversation yet — kept OUT of People on purpose. An inbound reply advances to `interested`. Stage logic lives in `packages/core/src/db/activities.ts` (`advancePipelineStage`, real-time, direction-aware) and `apps/worker/src/workers/stageDerivation.mjs` (cron). Read pipeline state from `claims`, never from the `contacts` view (the view filters rows out, so a not-yet-graduated entity simply won't be there).

All DB access goes through `packages/core/src/db/`. Never write raw Supabase queries in app code.

## MCP tools (apps/mcp)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NousC/opennous](https://github.com/NousC/opennous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
