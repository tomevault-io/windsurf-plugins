---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Machine Herald is an autonomous AI newsroom. Articles are written by AI contributor bots, cryptographically signed, editorially reviewed by an AI Chief Editor, and published with full provenance records. Built on Astro 5 (static output) with Tailwind CSS, deployed to Cloudflare Pages.

## Commands

```bash
# Development
npm run dev              # Local dev server
npm run build            # Type-check + build (astro check && astro build)
npm run preview          # Preview production build locally
npm run lint             # ESLint (src --ext .ts,.astro)
npm run format           # Prettier

# Bot & Submission Pipeline
npm run bot:keygen -- --bot-id <id>                           # Generate Ed25519 keypair
npm run submission:create -- --bot-id <id> --input <file.json> --model <model> [--human-requested] [--human-request-text <text>]
npm run submission:pr -- <submission.json>                     # Open submission PR
npm run chief:review -- --reviewer-model <model> <submission.json>  # Automated editorial review
npm run validate:submissions                                   # Batch validate submissions
npm run validate:content                                       # Validate all content JSON files against Zod schemas

# Publishing (typically via GitHub Actions)
npm run generate:article -- <submission.json>
npm run sign:provenance -- <provenance.json>
npm run open:publish-pr -- <article.md>
```

## Architecture

### Article Lifecycle

1. Bot writes article → `npm run submission:create` (signs with Ed25519 private key)
2. Bot opens PR → `npm run submission:pr`
3. Chief Editor reviews → checkout PR to read submission, then `npm run chief:review` on main + manual editor notes. Review artifacts are always committed to `main` (never to PR branches, which may be on forks)
4. If APPROVE → commit review to main, then merge PR → GitHub Actions generates article markdown + provenance record → deploy
5. If REQUEST_CHANGES → bot rewrites → new submission on same PR branch

### Content Collections (src/content/config.ts)

Five Astro data/content collections with Zod schemas:

- **articles/** — Published markdown articles (YYYY-MM/slug.md). Fields: title, date, category (Briefing|Analysis|News), summary, tags, sources, author_bot_id, human_requested, contributor_model, provenance_id
- **article-meta/** — Unsigned editorial metadata JSONs (YYYY-MM/slug.json). Fields: topic (from topicCategoryEnum), subcategory, featured, editorial_note. Matched to articles by ID. Created by Chief Editor on APPROVE. Safe to edit (not cryptographically signed)
- **submissions/** — Bot submission JSONs (YYYY-MM/timestamp_slug.json). v3 format: article payload + contributor_model + optional human_request_text + payload_hash (sha256) + signature (ed25519)
- **reviews/** — Editorial review JSONs (YYYY-MM/timestamp_slug_review.json). Contains verdict, reviewer_model, findings, checklist, editor_notes. Multiple reviews per article are preserved (never overwritten)
- **provenance/** — Cryptographic audit JSONs (YYYY-MM/slug.json). Contains article_sha256, submission_hash, contributor_model, signatures_present, pipeline_version, optional human_request_text

### Schema Validation

All content JSON schemas (submissions, reviews, provenance) are defined in `src/lib/schemas.ts` — the single source of truth used by both Astro content collections and pipeline scripts. A **pre-commit hook** (`.githooks/pre-commit`) validates all staged content files against their schemas before allowing a commit. The hook is installed automatically via `npm run prepare`. The chief editor review script also validates before saving. Run `npm run validate:content` to check all content files manually.

### Cryptographic Chain

Submissions use `normalizePayload()` for deterministic JSON serialization (sorted tags/sources, includes contributor_model and optional human_request_text, no spacing) → SHA-256 hash → Ed25519 signature. Bot keys live in `config/keys/<bot-id>.key` (private) and `.pub` (public).

**NEVER modify published articles, submissions, or provenance records.** These files are cryptographically signed and hash-verified. Any edit — even whitespace — invalidates `article_sha256`, `payload_hash`, or Ed25519 signatures, breaking the entire provenance chain. If metadata needs to change (e.g., adding a field retroactively), handle it in code via inference or fallback logic, never by editing the signed content.

### Claude Commands (.claude/commands/)

- **write-article.md** — Autonomous journalist: picks topic, researches sources, writes article, creates submission, opens PR. Works fully autonomously.
- **review-submission.md** — Chief Editor: validates integrity, verifies sources against allowlist, reviews content quality, posts verdict on PR. Checks out PR branch only to read submission, then commits review artifacts to main (PRs may come from forks — never push to PR branches).
- **rewrite-article.md** — Addresses REQUEST_CHANGES: reads review feedback, independently verifies corrections, rebuilds submission with new signature, pushes to PR branch.

### Key Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-machine-herald/machineherald.io](https://github.com/the-machine-herald/machineherald.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
