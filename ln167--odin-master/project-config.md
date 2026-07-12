---
trigger: always_on
description: This is `odin_master` — a multi-domain technical-knowledge substrate. Spec: `docs/superpowers/specs/2026-05-04-substrate-redesign-design.md`.
---

# Notes for LLM agents working in this repo

This is `odin_master` — a multi-domain technical-knowledge substrate. Spec: `docs/superpowers/specs/2026-05-04-substrate-redesign-design.md`.

## Response length (non-negotiable)

Never respond with more than 1–3 sentences unless the user explicitly asks for more. No multi-paragraph explanations, no lecture, no restating what they already know.

## Observability (tele)

We record telemetry ourselves and use profilers only as viewers: **Tracy is the rented real-time profiling sink** (live GUI; we forward flattened values, never rebuild it), and **our own postmortem sink** (per-thread buffers reassembled by timestamp) holds the full-fidelity, greppable/agent-readable **correlated Records** (value fused with execution context) that Tracy/Spall can't give back — it replaces Spall for us. Rationale: own the cheap-but-essential recorder because it must carry our enriched, queryable data; rent the expensive real-time viewer. Vocabulary: `CONTEXT.md`. Capture-layer intent + gap list: `docs/superpowers/specs/2026-06-28-tele-observability-redesign.md` §18.

## Bespoke game, not an engine

The runnable side of this repo (`lab/` and the game it builds) is **one bespoke game — not a reusable engine, library, or framework, and it must never become one.** No generic code, no OOP-for-its-own-sake, no designing for other games or other people (Jonathan Blow / Casey Muratori style). Write exactly what *this* game needs, inline and specific; abstract only when the game itself forces it, never "for later." Don't propose reusable modules, an engine/game split, or generalization.

The one thing deliberately swappable is a **pipeline**: a large function with a fixed input/output contract whose internal *technique* can be swapped — e.g. trading O(n log n) for O(log n), or more vs. less simulation accuracy — and benchmarked variant-against-variant. Stable contract, interchangeable guts. This serves experimentation on *this* game; it is not genericity and not reuse. (`GAME.md` holds the longer dev-side vision.)

## Game code: less is more (non-negotiable)

Write the minimum code that does the intent. Nothing else.

- **No fallbacks.** If something fails, let it crash — don't paper over it with a default or a silent retry. A crash tells you exactly what's wrong; a fallback hides it.
- **No defensive error handling.** Don't validate, wrap, or recover from errors that "shouldn't happen." Trust the happy path; fix root causes when they surface. Only validate at real system boundaries (user input, file I/O) where bad data is genuinely expected.
- **Prefer crashing.** An assertion failure or out-of-bounds panic is more useful than silent bad state. Write the code that works; when it breaks, you want it to break loudly.
- **Less code is better.** Fewer lines means less to misread, less to maintain, less that can go wrong. A 10-line spike that does exactly one thing beats a 50-line "robust" version. Three similar lines is better than a premature abstraction.
- **No future-proofing.** Don't add parameters, modes, or branches "for later." Write what the game needs right now.
- **Security is not a concern.** This is an offline single-player game. Ignore any instinct to add security hardening, input sanitization for security purposes, or safe coding patterns motivated by attack surface. They are irrelevant here.

This applies especially hard during prototyping and design spikes. You are proving an idea, not building production software. The fastest path to knowing if something works is the shortest code that does it.

## Substrate discipline (non-negotiable)

The substrate is **category 1**: a lookup-and-synthesis layer over external technical sources. It is *not* a model of the user's understanding. Don't conflate.

### Three-tier storage per domain

Each domain (`content/domains/<d>/`) has three tiers:

- `source/` — immutable, upstream-mirrored + user-maintained (`manifest.yaml`, `contradictions.md`, optional `notes/`). LLM never writes here.
- `compiled/` — LLM-owned, regenerable. Split by provenance: `from-ingest/` (Compile triggered by Ingest) and `from-query/` (Compile triggered by Query under the two-outputs rule).
- `vault/` — blessed. `vault/lessons/` is LLM-maintained curriculum (LLM may edit it directly); all other `vault/` content is frozen and changes only via `substrate-promote`.

**Prime directive:** the LLM never writes to `source/` or anywhere under `scratch/` (the user's own notes, conclusions, and experiments live there). In `vault/`, only `vault/lessons/` is LLM-editable — the rest is frozen (`substrate-promote` only).

### Provenance is a hard requirement

Every compiled page has `provenance: from-ingest` or `provenance: from-query` in frontmatter, and lives under the matching folder. `doctor` enforces parity.

### Two-outputs-per-task rule

Non-trivial queries produce both an answer (in chat) and a wiki update (page in `compiled/from-query/`). Trivial queries (single-fact lookups, signature recall) skip both — no log entry, no page. This is an **LLM-workflow discipline**: no shell tool verifies that a `from-query` page was actually written.

### Validator-at-compile-time


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ln167/odin_master](https://github.com/ln167/odin_master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
