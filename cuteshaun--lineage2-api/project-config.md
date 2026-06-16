---
trigger: always_on
description: This is a public, read-only community API for Lineage 2 Interlude.
---

# Project Context: Lineage 2 API

## Overview

This is a public, read-only community API for Lineage 2 Interlude.

The architecture is chronicle-aware and designed to support additional chronicles in the future.

The project is unofficial and not affiliated with NCSoft.

The API, landing page, and documentation site are deployed in production on Vercel.

---

## Current Production Status

- The API is live in production.
- The landing page is live in production.
- The documentation site (Astro / Starlight) is live in production.
- The knowledge base / explorer example is live in production as a separate repo.

Three production-facing surfaces:

1. **API + landing** — this repo (`main`)
2. **Astro / Starlight docs** — `apps/docs/` in this repo
3. **Knowledge base / explorer example** — a separate repo, deployed independently

The API remains the source of truth. The landing, docs, and knowledge base are adoption and distribution surfaces; they consume the public API contract, they do not redefine it.

Public API contract is frozen unless a release-blocking issue is found.

### Still in progress

- Desktop design for the public-facing surfaces is mostly done.
- Mobile design is not finished.
- SEO is incomplete: metadata, titles, descriptions, Open Graph tags, favicon, and social-preview images.
- Documentation exists but is still raw and needs more real examples.
- Analytics are not installed yet.

---

## Repository Layout

### main (this repo)

`main` ships the backend API, the public landing page, and the documentation site.

It is the canonical source for:

- data parsing
- generated JSON output
- public API routes
- raw API routes
- DTO mapping
- API contract documentation
- snapshot tests
- build-time enrichment
- the public landing page (`src/app/page.tsx`, `src/app/about/page.tsx`)
- the Starlight docs site (`apps/docs/`)

`main` is what's deployed at the public API + landing URL and at the docs subdomain.

Do not add database-explorer UI, user-facing wiki pages, comments, ratings, screenshots, or community features to this repo. Those belong in the separate knowledge-base / explorer repo described below.

### Knowledge base / explorer (separate repo)

A reference client and visual sandbox for the public API now lives in **its own deployed repo**, not in this one. The old `ui-explorer` branch has been moved out and is no longer the primary location for explorer work.

The knowledge base / explorer exists to:

- dogfood the public API endpoints
- expose missing or awkward API relations
- demonstrate real player-facing use cases
- inform API improvements via real consumer feedback

The knowledge base / explorer must not become the source of truth. The API is the source of truth.

If the explorer needs heavy client-side workarounds, first consider whether the DTO should be improved in this repo. Any API change motivated by explorer use should remain useful to other external consumers (see *API Changes from External Clients* below).

---

## Data Source & Build Model

- Data is parsed from aCis datapack XML files
- Selected client DAT metadata may be parsed at build time when it adds clear public value
- Data is generated during build (`scripts/build-data.ts`)
- Output is stored as JSON under `data/generated/{chronicle}`
- The API reads from generated JSON at runtime
- No direct XML or DAT parsing in production

Generated files under `data/generated/{chronicle}` must not be hand-edited. Fix parsers, build scripts, DTO mapping, or UI consumption instead.

---

## Product Philosophy

Public endpoints must be:

- clean
- deduplicated
- stable
- player-friendly

Raw endpoints must:

- preserve original data structure
- stay close to engine truth

Prefer:

- additive changes
- backward compatibility
- minimal abstraction

---

## Current Data Model

The API ships the following domains (v1 release candidate; public API contract is frozen unless a release-blocking issue is found):

- Items
- NPCs / Monsters (cleaned + raw layers)
- Drops and Spoil, both directions
- Spawns (cleaned, with region + location enrichment; raw stays unenriched)
- Recipes
- Skills
- Armor sets
- Classes (with skill-learn tables and spellbook refs)
- Spellbooks (forward + reverse links between item ↔ skill ↔ class)
- Shops: merchant `buyLists`, curated multisells, NPC `/shop` view
- Quests (with optional client journal entries from `questname-e.dat`)
- Regions (engine death-teleport regions from `mapRegions.xml`)
- Locations (player-facing hunting zones from `huntingzone-e.dat`)
- Hennas (joined from `hennas.xml` + `hennagrp-e.dat`)
- OpenAPI spec at `/api/openapi.json` (chronicle-agnostic)

### Key Decisions

- NPC canonical identity uses:
  - `(name, level)`

- Drop system:
  - exact duplicates collapsed via `rollCount`
  - Adena (`itemId=57`) always has `type="adena"`
  - sorted for readability, not raw category order
  - reward cross-link `rewardedByQuests` carries per-quest counts;
    Adena rewards surfaced via `q.rewards.adena` scalar (not `items[]`)

- Skills:
  - primarily parsed from aCis XML
  - indexed by `"id-level"`
  - enriched at build time with selected client metadata when useful
  - may include `description: string | null` when available
  - `itemSkill` is resolved into DTO summaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cuteshaun/lineage2-api](https://github.com/cuteshaun/lineage2-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
