---
trigger: always_on
description: Always-on guardrails for any Claude Code session in this repo. Procedural workflows live in `.claude/skills/`; this file only encodes the invariants that protect the catalogue from accidental damage.
---

# CLAUDE.md

Always-on guardrails for any Claude Code session in this repo. Procedural workflows live in `.claude/skills/`; this file only encodes the invariants that protect the catalogue from accidental damage.

## What this repo is

An open-source Business Architecture Reference Catalogue with **three orthogonal artefacts**: capabilities (BC) describe WHAT, business processes (BP) describe HOW, value streams (VS) describe end-to-end value delivery. YAML files in `catalogue/` are the **single source of truth**. Everything else (`dist/api/*.json`, the bundled Python package data, the Astro site) is built from those YAML files.

## Invariants — capabilities (BC-)

- **Source of truth:** edit only `catalogue/*.yaml` and `schema/capability.schema.json`. Never hand-edit `dist/api/**` or `packages/py/src/turbo_ea_capabilities/data/**`. Both are build artefacts and are wiped by the next `npm run build`.
- **ID format:** `BC-<L1>[.<L2>[.<L3>[.<L4>]]]`. Max depth is **L4**. If L5 feels needed, the model has slipped into process territory — push it into the process layer (`BP-` ids).
- **MECE:** within any parent, children must be Mutually Exclusive and Collectively Exhaustive.
- **Names are noun phrases:** Title Case, 2–5 words, no verbs, no articles, no vendor / product / org / geography names, no value-stream names (e.g. *Order-to-Cash* is a value stream, not a capability).
- **Sparse numbering:** new siblings use 10, 20, 30, … to leave room for inserts. Retired IDs are **never reused**.
- **Single parent:** a capability has exactly one parent. For multi-use, model a *shared service* relationship — not a multi-parent edge.
- **Deprecation:** `deprecated: true` requires `deprecation_reason` and (when applicable) `successor_id`.
- **Industry tag:** `Cross-Industry`, a single industry name, or `;`-separated list. L2+ inherits from L1 unless overridden.

## Invariants — value streams (VS-)

- **Source of truth:** `catalogue/_value-streams.yaml` (single file). Schema: `schema/value-stream.schema.json`.
- **Stream id:** `VS-<n>` sparse 10/20/30. Names use the bookend pattern `<Trigger>-to-<Outcome>`.
- **Stage id:** `VS-<n>.<m>` sparse 10/20/30. Stable across reorderings — `stage_order` is the visual rank, not part of identity.
- **Stage `capability_ids` are L1 only.** Use `notes` to capture sub-scope detail. Stage `process_ids` may resolve at any BP depth.
- Industry vocabulary inherited from BC L1s. `Cross-Industry` must stand alone.

## Invariants — business processes (BP-)

- **Source of truth:** `catalogue/processes/BP1-<slug>.yaml` (one file per Category), indexed in `catalogue/processes/_index.yaml`. Schema: `schema/business-process.schema.json`.
- **ID format:** `BP-<L1>[.<L2>[.<L3>[.<L4>]]]` mirroring BC. Cross-Industry BP1s live in the sparse `BP-1000`..`BP-1160` range (one BP1 per Cross-Industry value stream, generated from BC + VS); industry-specific BP1s use `BP-130`..`BP-490`. Max depth **L4** (Category → Group → Process → Activity); Cross-Industry BPs stop at L3 by construction. BPMN-level steps belong in diagrams, not the catalogue.
- **Naming is two-tier:** BP1 roots aligned to a value stream use the VS bookend name verbatim (*Order-to-Cash*, *Hire-to-Retire*, *Procure-to-Pay*). BP2/BP3/BP4 names are verb-phrased operational activities (*Capture Customer Order*, *Verify Customer Credit*, *Allocate Inventory to Order*). One canonical name per node; **aliases are disabled by governance** — do not author them.
- **Industry tag:** same scheme as capabilities. BC L1 industry vocabulary is the master list.
- **`realizes_capability_ids`** is the single source of truth for the BC↔BP link; the reverse `Capability.realizes_processes` is derived at build time.
- **`framework_refs`** for structured cross-walks to APQC-PCF, BIAN, eTOM, ITIL, SCOR, DCOR, COBIT, SHRM-BoCK, ISO-55000, ISO-31000, COSO-ERM, TOGAF, BIZBOK, ACORD, ICMM. Cross-Industry BPs are no longer authored from APQC PCF — they are generated from BC + VS — but APQC-PCF can still be cited as a secondary cross-walk where structurally honest. Industry-specific BPs anchor on their domain framework (BIAN for banking, eTOM for telco, ACORD for insurance, ICMM for mining, the relevant APQC industry PCF, etc.). Used alongside (not instead of) the free-form `references[]` URI list.

## Invariants — macro capabilities (MC-)

- **Source of truth:** `catalogue/_macro-capabilities.yaml` (single file). Schema: `schema/macro-capability.schema.json`.
- **Purpose:** an executive navigation overlay above L1 — a separate, orthogonal artefact (same pattern as value streams). Does **not** enter the BC tree; does **not** participate in VS / BP links; does **not** change any L1 id, level, or file.
- **ID format:** `MC-<n>` with sparse 10/20/30 numbering. Stable; never reused.
- **Naming:** Title Case noun phrase, 2–5 words, no verbs, no articles. Same rules as capability names.
- **`capability_ids` is L1 only.** Plain `BC-<N>`, no dots. Sub-scope detail lives in the L1, not the macro.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentmakes/turbo-ea-capabilities](https://github.com/vincentmakes/turbo-ea-capabilities) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
