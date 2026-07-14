---
trigger: always_on
description: **QECirc** is a community-driven web library for quantum error correction (QEC) circuits.
---

# CLAUDE.md

## Project Overview

**QECirc** is a community-driven web library for quantum error correction (QEC) circuits.
Users can browse and discover circuits, and contribute new ones by opening a GitHub Issue.
Circuits are stored in STIM format and converted to QASM/Cirq for display.

---

## Data Model & Hierarchy

```
Code                          e.g. "Surface Code"
  └── Circuit                     e.g. "Standard Encoding"
```

Circuits belong directly to codes. Circuit type (e.g. `encoding`, `syndrome-extraction`)
is represented as a tag, not a separate entity.

Both levels support **tags** to aid discovery and filtering:

| Level   | Example tags                               |
| ------- | ------------------------------------------ |
| Code    | `CSS`, `topological`, `bosonic`            |
| Circuit | `encoding`, `fault-tolerant`, `distance:3` |

Tags can be either **structured** (`key:value`, e.g. `distance:3`) or **free-form strings**.

Circuits also have numeric **metrics** for filtering: `gate_count`, `depth`, `qubit_count`.

### Database Schema

```
codes
  id, name, slug, n, k, d, zoo_url,
  h, logical, canonical_hash, created_at
  -- n, k, d: code parameters [[n,k,d]] for direct querying/sorting
  -- zoo_url: optional link to QEC Zoo
  -- h: symplectic stabilizer matrix, shape (n−k) × 2n, JSON-encoded
  -- logical: symplectic logical operators, shape 2k × 2n, JSON-encoded
  -- For CSS codes, the Hx/Hz/Lx/Lz view is derived in the UI via splitHToCss
  -- canonical_hash: SHA256 of canonical form for dedup (indexed)

tools
  id, name, slug, description, homepage_url, github_url, paper_urls, created_at
  -- software tools used to create circuits
  -- paper_urls: JSON-encoded array of associated paper URLs

circuits
  id, qec_id, code_id → codes, name, slug, notes, source,
  gate_count, two_qubit_gate_count, depth, qubit_count,
  crumble_url, quirk_url, tool_id → tools, created_at
  -- qec_id: permanent globally unique circuit identifier (displayed as #N, never reused)
  -- source: provenance (DOI, URL, or citation)
  -- gate_count, two_qubit_gate_count, depth, qubit_count: numeric metrics for filtering
  -- crumble_url, quirk_url: optional external visualization links
  -- tool_id: optional link to tool used to create the circuit

circuit_bodies
  id, circuit_id → circuits, format, body
  -- format: circuit format identifier (e.g. 'stim', 'qasm', 'cirq')
  -- UNIQUE(circuit_id, format): one body per format per circuit

circuit_originals
  id, circuit_id → circuits (UNIQUE),
  original_stim, original_h, original_logical
  -- pre-canonicalization data as submitted by contributors
  -- matrix fields are JSON-encoded (same format as codes.h / codes.logical)
  -- populated from data_yaml/circuits/originals/

tags
  id, name                          -- e.g. "CSS", "distance:3", "encoding"

taggings
  tag_id → tags, taggable_id, taggable_type  -- taggable_type ∈ {code, circuit, tool}
  -- composite PK (tag_id, taggable_id, taggable_type)
```

---

## Circuit Format

Circuits are stored in STIM format and converted to QASM/Cirq for display.
The STIM body is the canonical source; QASM/Cirq are generated as alternate
views in `circuit_bodies`.

---

## Submission Workflow

Circuits are submitted by opening a GitHub Issue using the provided template.
A maintainer reviews the issue, then uses the ingestion pipeline to add the circuit.

- **[docs/adding-circuits-agent.md](docs/adding-circuits-agent.md)** — agent-assisted workflow (`/add-circuit` command)
- **[docs/adding-circuits.md](docs/adding-circuits.md)** — manual workflow (Python scripts + YAML editing)

---

## Tech Stack

| Layer     | Choice                      | Rationale                                      |
| --------- | --------------------------- | ---------------------------------------------- |
| Framework | Astro v7 (TypeScript)       | Static-first with SSR opt-in for dynamic pages |
| Database  | SQLite via `better-sqlite3` | Zero external services, file-based, simple     |
| Styling   | Tailwind CSS                | Standard utility-first, minimal custom CSS     |
| Hosting   | Self-hosted (agnostic)      | Avoid platform lock-in                         |

**Rendering strategy — Astro v7 (static default, SSR opt-in):**

- Static pages: landing page, 404 (pre-rendered at build time)
- SSR pages (`prerender = false`): all `/codes/...` and `/circuits/...` routes, `/api/search` (rendered on request, read from SQLite)
- Client-side JS: search bar (debounced fetch), circuit row expand/collapse, format switching, favorites (toggle/filter/export/import), CodeBlock copy/download, lazy-loaded circuit bodies on code pages (fetched from `/api/circuits/[qec_id]/bodies` on first row expand), and filtering/sorting on the listing pages (`list-filter-client.ts` over `data-metrics`/`data-tags` row attributes — the server always renders the canonical full list and ignores filter params; `/api/download` still parses them)

This keeps the site fast and simple while scaling comfortably to thousands of circuits.

**SEO & AI discoverability** (goal: be cited/linked by search and AI answer engines, without inviting training crawlers):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qecirc/qecirc-website](https://github.com/qecirc/qecirc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
