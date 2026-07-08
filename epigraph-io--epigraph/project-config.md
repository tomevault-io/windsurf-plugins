---
trigger: always_on
description: This file is loaded into any Claude Code / agent session opened in this repo.
---

# EpiGraph — Claude / Agent Conventions

This file is loaded into any Claude Code / agent session opened in this repo.
Project-wide rules below; module-specific rules live next to their code.

## Retiring backlog items

When you complete or refute a claim labelled `backlog`, **always use
`mcp__epigraph__resolve_backlog_item(original_id, resolution_content)`**.
It creates a resolution claim (labelled `["resolved"]`, prose prefixed
`"Resolves <id>: "`) AND patches the original's labels with `add=["resolved"]`
in a single call. Free-text "Resolves <UUID>" alone leaves the original
looking open in every backlog query forever.

Do NOT:
- File a free-text "Resolves <UUID>" claim alone without patching the
  original's labels.
- Use `supersedes`/`is_current` for status. Those are reserved for
  *epistemic* claim replacement (one claim refining another's factual
  content), not operational status.
- Reach for raw `update_labels` to add `["resolved"]` to a backlog item —
  that bypasses the canonical resolution-claim trail.

**Querying open backlog:**

```python
mcp__epigraph__query_claims_by_label(
    labels=["backlog"],
    exclude_labels=["resolved"],
    current_only=True,
)
```

A daily reconciler (`scripts/reconcile_backlog_labels.py`) catches free-text
"Resolves <UUID>" claims filed without `resolve_backlog_item` and back-fills
the label patch. Ambiguous matches go to `docs/superpowers/reports/reconciler-needs-review.log`.

Full spec: `docs/conventions/backlog-retirement.md`.

## Schema, migrations, claim mechanics

- All SQL stays in `crates/epigraph-db/src/repos/`. HTTP routes
  (`crates/epigraph-api/src/routes/`) and MCP tools
  (`crates/epigraph-mcp/src/tools/`) both call the repo layer; do not
  duplicate SQL between them.
- After adding or modifying a `sqlx::query!` / `sqlx::query_as!` macro
  call, run `DATABASE_URL=... cargo sqlx prepare --workspace -- --tests`
  and commit `.sqlx/` so `SQLX_OFFLINE=true cargo check --workspace`
  passes in CI.
- `claim_from_row` has ~20 callers in `crates/epigraph-db/src/repos/claim.rs`.
  Do not widen its signature — extend the relevant `SELECT` in the caller
  and post-fix the returned `Claim` (see `list_by_labels` and `get_by_id`
  for the pattern).

## Test database

Integration tests against the live `epigraph` DB fan out for 30+ minutes
and pollute production claim state. Use `epigraph_db_repo_test` (or any
small DB):

```bash
DATABASE_URL=postgres://epigraph:epigraph@localhost/epigraph_db_repo_test cargo test ...
```

## Workflow

- Feature branches, never land 3+ commits directly on `main`.
- `gh pr merge --merge --delete-branch` by default; never `--squash`
  unless explicitly told.

## Embedding policy

**Invariant:** every **non-telemetry** claim with `is_current = true` should
have an embedding; every claim with `is_current = false` should have
`embedding = NULL`. Semantic recall (`recall()`, `recall_with_context()`,
`theme_cluster`, `find_workflow`'s semantic path) reads from `embedding`, so
violations either hide live claims or surface stale ones.

**Telemetry exception:** host-provenance claims (epiclaw-host's
`ProvenanceRecorder` — container/task lifecycle, agent output, messages) are
intentionally NOT embedded (no semantic value, one OpenAI call each). They carry
the `telemetry` label and a `properties->>'event'` marker, and dominate the
is_current embedding gap. The write path already skips embedding them
(`submit.rs` `is_host_telemetry`), and `find_claims_needing_embeddings` excludes
them. Do NOT treat them as `live_missing`. (backlog a4aaa487)

### Write paths (must embed on insert)

When adding a new code path that inserts a claim, embed inline post-commit,
best-effort (warn on failure, never block the write). Current call-sites:

- **MCP `submit_claim`** — `crates/epigraph-mcp/src/tools/claims.rs:217`
- **MCP `memorize`** — `crates/epigraph-mcp/src/tools/memory.rs:103`
- **MCP `batch_submit_claims`** — delegates to `submit_claim`
- **MCP `ingest_document`** — `crates/epigraph-mcp/src/tools/ingestion.rs:321`
- **MCP `ingest_document_spine`** — `do_ingest_document_spine` in the same file; phase-1 of the two-phase flow, embeds spine claims (thesis + sections + paragraphs) post-write
- **MCP `workflow_ingest`** — embeds executor output; `crates/epigraph-mcp/src/tools/workflow_ingest.rs`
- **MCP `store_workflow`** — embeds executor output via `execute_workflow_ingest_with_inserted`; `crates/epigraph-mcp/src/tools/workflows.rs::store_workflow`
- **MCP `improve_workflow_hierarchy`** — embeds executor output (claims) AND calls `set_goal_embedding` for the new `workflows` row; `crates/epigraph-mcp/src/tools/workflow_ingest.rs`
- **MCP `add_step`** — embeds when `AddStepResult::inserted_content` is `Some`
- **HTTP `POST /api/v1/claims`** — `crates/epigraph-api/src/routes/claims.rs` (after `tx.commit()` in `create_claim`)
- **HTTP `POST /api/v1/submit/packet`** — `crates/epigraph-api/src/routes/submit.rs:1480`
- **HTTP `POST /api/v1/workflows/ingest`** (both callsites) — `crates/epigraph-api/src/routes/workflows.rs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epigraph-io/epigraph](https://github.com/epigraph-io/epigraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
