---
trigger: always_on
description: This file provides guidance to coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working with code in this repository.

## Repo state

Pre-alpha. Most code referenced in `docs/MVP_SPEC.md` doesn't exist yet — it's split into ~49 child issues across 15 epics in Project #7. Verify file tree before assuming.

## Canonical references

- `docs/MVP_SPEC.md` — v0 scope. Cite section numbers when scope is in question.
- `docs/ARCHITECTURE.md` — current technical realization (stack, lifecycle, storage, invariants). Read before designing anything cross-component.
- `docs/BRAND_FOUNDATIONS.md` — voice, naming, positioning.
- `docs/METHODOLOGY.md` — autonomy tiers (low/medium/high).
- `docs/GROOMING_RUNBOOK.md` — operating the `backlog_grooming` loop: invocation, what approving actually does (it writes), the autonomy-tier policy for this repo (`autonomy:low` means an agent STRUCTURALLY cannot, not that the work is sensitive), and turning the ranking into a campaign.
- `docs/spec/` — canonical JSON Schemas + reference docs for the workflow spec (`.fishhawk/workflows.yaml`) and the plan artifact (`standard_v1`). Validate with `check-jsonschema --schemafile <schema> <yaml-or-json>`. Caveat: a workflow-v2 document using `defaults`/`extends` must be validated with `fishhawk validate` (or the backend), not a bare `check-jsonschema` run — the bare run resolves no same-document reuse, so it rejects a reuse-bearing spec (e.g. a stage inheriting its `executor`) the product accepts (#2340). To keep `check-jsonschema` in the loop anyway, pipe it the resolved document: `fishhawk validate --emit-resolved <yaml> | check-jsonschema --schemafile docs/spec/workflow-v2.schema.json --default-filetype yaml -` (E52.22 / #2351) — `--default-filetype yaml` is required because check-jsonschema assumes JSON on stdin, and emit RESOLVES without VALIDATING (no charter rule, no stage-reference sweeps), so `fishhawk validate` without the flag stays the authority.
- `docs/api/` — REST API surface: `v0.openapi.yaml` is source of truth, `v0.md` is the human companion. Lint with `npx -y @redocly/cli@2.31.5 lint --config docs/api/redocly.yaml docs/api/v0.openapi.yaml` (pinned version — see the pinning rule under "Build, test, lint").
- `.fishhawk/workflows.yaml` — placeholder; executed by the product itself starting Day 21 (~2026-05-20).

## Documentation surfaces

- `README.md` (root) is the only human-facing doc. Keep narrative.
- Everything in `docs/` is agent-consumed. Write structured, dense, no fluff.
- **`docs/ARCHITECTURE.md` §10 "Where to look" rows are pointers, not prose**: `path + ≤1 sentence + one anchor issue/ADR ref`, linking a package `README.md` for anything longer. The long-form behavioral contract, invariants, and issue history for a package live in that package's `README.md`, next to the code — an agent touching the package reads them in context instead of every agent paying for all of them up front. `scripts/test verify` fails any ARCHITECTURE.md line over 1000 chars (fat single-line rows are silently truncated by agents' file-read tooling, which cuts lines around 2000 chars).
- **`site/` is the public documentation site** (E12.1 / #2261): human-facing narrative, Astro + Starlight, source path `site/`, published to GitHub Pages at <https://kuhlman-labs.github.io/fishhawk/>. The reciprocal rule is what makes the split structural rather than conventional — `site/` is human-facing narrative and `docs/` stays agent-consumed and is **never a publish source**; nothing under `site/` imports, globs or symlinks `docs/` (prose cross-links to `docs/` as GitHub URLs are fine and expected). Long-form contract, dependency pins and the local loop: `site/README.md`.
- **The site is UNVERSIONED and tracks `main`; workflow-spec majors are carried as CONTENT, not as site versions** — a support table on `site/src/content/docs/reference/index.md` plus **ONE** `reference/workflow-spec.md` covering v0, v1 and v2 with the differences inline, NOT one page per major. This mirrors `docs/spec/workflow-v2.md`, which is the complete standalone reference for the live major while v0/v1 are frozen; per-major pages would mirror a structure the canonical reference deliberately abandoned and would signal that authoring a new v0 spec is a live choice the product discourages. Decided in #2261 — do not re-litigate; both revisit triggers (numbered releases ship; a future major diverges past inline readability) are recorded on `site/src/content/docs/reference/versioning.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kuhlman-labs/fishhawk](https://github.com/kuhlman-labs/fishhawk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
