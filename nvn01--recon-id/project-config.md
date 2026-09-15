---
trigger: always_on
description: Recon is an early-stage repository for building the easiest way to monitor wishlist computers, computer parts, and peripherals across multiple preloved marketplaces and social platforms.
---

# Recon Agent Instructions

## Project Purpose

Recon is an early-stage repository for building the easiest way to monitor wishlist computers, computer parts, and peripherals across multiple preloved marketplaces and social platforms.

The product goal is simple: users should not need to repeatedly check every marketplace by hand. Recon should discover relevant new listings quickly, normalize them into a consistent structure, and make them easy to inspect, compare, and revisit.

## Current Status

Recon v1 entered production on 2026-07-16. The public discovery UI, read-only
API, PostgreSQL schema, multi-source collector, centralized NVIDIA AI manager,
independent Instagram R2 media worker, and direct production ingestion path are
live. Treat the current repository as a working production system, not an
exploratory scaffold.

The main operational focus now is:

- Make scraping repeatable, source-aware, and resilient.
- Normalize listings into a consistent data structure before writing them to the database.
- Extract useful database fields from messy post descriptions, including category, brand, price, condition, locations, status, and seller context.
- Avoid rate limits, temporary blocks, duplicate spam, and brittle scraping behavior.
- Keep the system inspectable so future agents can understand what happened during each scrape run.
- Preserve the split production topology and deploy only fixed semantic-version images.
- Monitor connector health, AI queue throughput, production disk usage, and database growth.

Phase 5 operational hardening is implemented. Every connector has a sanitized
parser fixture, duplicate-run locks and connector cooldowns have regression
coverage, and `scraper.operational_report` produces separate daily data-quality
and manual-review JSON artifacts under the persisted scraper log volume.

The public UI and backend contracts are implemented. Keep scraper operations,
database writes, and public UI concerns separated unless a requested change
explicitly crosses those boundaries.

## Product Freshness Goal

The long-term product goal is near minute-level freshness so users do not miss wishlist items.

Do not blindly run every connector every minute in production. Prove source safety first. Start with conservative staging checks, connector-level cooldowns, retry limits, backoff, and clear health metrics. Tighten cadence only when the source can tolerate it without lockouts, noisy failures, or duplicate pileups.

## Planning Context

Before making major changes, inspect the existing planning artifacts:

- `.plan/first-pass.html`
- Any nearby planning notes or scraper files related to the requested task.

Preserve the current architecture direction unless the user explicitly changes it:

1. Scraper and backend setup.
2. Database schema and ingestion contracts.
3. Scraper core and source connectors.
4. Backend API.
5. Operational hardening before public UI work.

## Major Feature Proposal Workflow

When Novandra opens a new branch/worktree or asks what a new major feature would look like, do not start building immediately.

First answer with a complete feature model/spec that explains:

- What the user experience should be.
- What data model or storage changes are needed.
- What API/backend behavior is needed.
- What frontend behavior is needed, if any.
- How the feature works without violating current project constraints, such as no user accounts or no authentication.
- Important tradeoffs, failure modes, abuse risks, privacy risks, and edge cases.
- How the feature should be tested and verified.
- Whether it belongs in the current plan or should update `.plan/first-pass.html`.

Example trigger:

> what would it look like if user interested in one post then want to save or like some the product in the web app without need to doing some login or authentication since our app is not have users profile feature yet.

For this kind of request, propose the whole model first. Wait for explicit approval before implementing. Approval may be phrased casually, such as "Oke i like that, build it!", "Love it, build it!", "build it", or a similar clear instruction.

If Novandra asks follow-up questions, keep refining the model/spec. Do not treat interest in the idea as permission to build.

## Private Local Skills

The `.agents/` directory is intentionally private and ignored by git. It may contain personal, sensitive, or non-public workflow knowledge.

When `.agents/` is available locally:

- Read `.agents/RECRUITED_SKILLS.md` before choosing workflow skills.
- Use only the relevant skill files for the task.
- Do not copy private skill contents into public repository files.
- Do not remove `.agents/` from `.gitignore`.
- Do not assume GitHub or another remote environment has access to `.agents/`.

When `.agents/` is not available, continue with the public repository context and explain any limitation if it affects the work.

## Skill And Agent Usage

Use specialized skills and subagents when they materially improve the work, especially for:

- Scraper design and connector investigation.
- Backend and API design.
- PostgreSQL, Prisma, and migration planning.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nvn01/Recon.id](https://github.com/nvn01/Recon.id) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
