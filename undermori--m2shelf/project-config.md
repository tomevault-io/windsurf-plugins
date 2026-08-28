---
trigger: always_on
description: This file contains stable rules for every coding agent that works in this repository. It is not a copy of any one delivery request.
---

# M²Shelf Agent Guide

This file contains stable rules for every coding agent that works in this repository. It is not a copy of any one delivery request.

## Read first

For a fresh account, machine, or agent, read in this order before changing code:

1. `AGENTS.md`
2. `docs/PRODUCT_SPEC.md` — intended product behavior
3. `docs/PROJECT_CONTEXT.md` — the implementation that exists now
4. `docs/DECISIONS.md` — decisions that must not be silently reversed
5. the user's current request
6. the code and migrations related to that request

Repository documents, code, and tests are the durable source of context. Do not assume access to an earlier chat, Codex Memory, or another agent's private state.

## Product identity

- User-facing brand: **M²Shelf**
- Technical / ASCII name: **M2Shelf**
- Subtitle: **MORI MEDIA SHELF**
- Product: a Windows, local-first media collection browser and manager
- Use `M²Shelf` in ordinary UI. `M2Shelf` is allowed for executable, archive, package, identifier, and other technical names.

## Non-negotiable product rules

- Treat every user media root as read-only. Never rename, move, delete, rewrite, or create files in it.
- A display-name change updates M²Shelf's SQLite data only; it never changes a real file or folder name.
- Bangumi data belongs to the metadata and presentation layer. It never renames source media.
- A completed scan or an explicit “match existing resources” action may auto-bind an unbound, Bangumi-eligible Node after structured title evidence, at most three official search queries, bounded multilingual candidate scoring, and bounded detail enrichment when needed. Bind the highest-scoring candidate once it reaches the configured direct threshold and has no strong season/year/edition/type conflict; there is no user-visible Pending tier or score-margin gate, and provider rank is the deterministic final tie-breaker. The first eligible result of the primary official query may be raised to the direct threshold but never bypasses a hard conflict or the Container exact-primary rule. Only Bangumi animation (Subject type 2) and live-action (Subject type 6) entries are eligible. A Container must additionally match its primary title exactly. Merge the bounded multi-query candidate pool fairly instead of letting an earlier query consume it. In one matching run, enrich at most the five strongest candidates per Node, reuse cached Subject details, and start no more than 256 new detail requests in total; when the remaining budget permits, reserve at least one uncached detail opportunity for each later Node, then continue from bounded search metadata after exhaustion. Never replace an existing binding or manual cover in the ordinary automatic path, and keep manual search available for correction. Only the user's explicit “rematch selected” action may replace an existing Bangumi binding after the same direct and hard-conflict gates; it still never replaces a manual cover. A non-manually-classified supplementary child such as SP/OVA/Extras under a parent with direct videos is structurally excluded from automatic candidates, while manual Bangumi binding remains available.
- An explicit manual Bangumi choice may store bounded, cleaned title evidence from that Node as application-owned confirmed aliases. Automatic bindings must never create confirmed aliases. A later automatic match may reuse the first evidence-priority alias whose own stored observations all agree on one supported Bangumi Subject; disagreement for that alias is ambiguous and must fall back to ordinary official search. Preserve meaningful season/year qualifiers in these aliases. The recalled Subject gets bounded official-detail priority, while ordinary official queries remain available if it is stale, conflicted, or unavailable. Confirmed aliases provide exact title evidence only and must not fabricate provider season/year/edition signals. Alias reuse still passes the current Node's hard-conflict and Container safeguards, never sends the alias to a new third-party service, and is removed with its source Node or when that source binding is cleared or replaced.
- Keep a valid Bangumi binding when cover retrieval fails, expose that failure, and allow an explicit retry. If an automatic cover was cleared or its cached file disappeared, an ordinary scan/match pass may restore only that cover from the retained binding URL; it must not search again, replace the binding, or override a manual cover.
- Store downloaded and manually selected covers only in the application-owned cache.
- Non-video files are resources attached to a Node. They do not become works, Bangumi candidates, or project-count entries merely because they exist.
- Work / Container / Mixed classification remains video-distribution based and must preserve manual overrides across rescans.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Undermori/M2Shelf](https://github.com/Undermori/M2Shelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
