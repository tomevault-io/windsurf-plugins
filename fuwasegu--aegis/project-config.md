---
trigger: always_on
description: Aegis is an MCP server that compiles deterministic context for AI coding agents.
---

# Aegis — DAG-based Deterministic Context Compiler

## Architecture

Aegis is an MCP server that compiles deterministic context for AI coding agents.
Instead of RAG (retrieval-augmented generation), it uses a DAG (directed acyclic graph)
of edges to resolve exactly which documents an agent needs for a given set of target files.

## Running

```bash
npm run build          # compile TypeScript
npm run start:agent    # start agent surface (default: ./aegis.db)
npm run start:admin    # start admin surface
npm test               # run all tests
```

## Agent Surface Tools (6 tools)

- `aegis_compile_context` — Primary tool. Given target_files, returns deterministic context.
- `aegis_observe` — Record observations (compile_miss, review_correction, pr_merged, manual_note, document_import, doc_gap_detected).
- `aegis_get_compile_audit` — Retrieve audit log of a past compile invocation.
- `aegis_get_known_tags` — List distinct intent tags (approved-resolvable) with `knowledge_version` and SHA-256 `tag_catalog_hash` for client caching.
- `aegis_workspace_status` — Read-only workspace snapshot (recent compile regions, unresolved compile_miss backlog, pending proposal count).
- `aegis_init_detect` — Analyze a project to generate initialization preview (read-only).

## Admin Surface Tools (18 additional tools, 24 total)

- `aegis_init_confirm` — Confirm initialization using preview_hash from init_detect.
- `aegis_list_proposals` / `aegis_get_proposal` — Review pending proposals.
- `aegis_approve_proposal` / `aegis_reject_proposal` — Approve or reject with optional modifications.
- `aegis_preflight_proposal_bundle` / `aegis_approve_proposal_bundle` — Validate or approve pending proposals sharing a `bundle_id`.
- `aegis_check_upgrade` — Check for template version upgrades.
- `aegis_apply_upgrade` — Generate proposals for template upgrade changes.
- `aegis_archive_observations` — Archive old observations.
- `aegis_get_stats` — Aggregate knowledge counts and health signals.
- `aegis_list_observations` — List observations with outcome-based filtering (proposed / skipped / pending).
- `aegis_import_doc` — Import a document into Canonical Knowledge (from `content` or `file_path`).
- `aegis_analyze_doc` / `aegis_analyze_import_batch` — ADR-015 import-plan analysis (read-only).
- `aegis_execute_import_plan` — Materialize analyzed import plans as proposal bundles.
- `aegis_process_observations` — Trigger observation analysis pipeline for pending observations.
- `aegis_sync_docs` — Synchronize imported documents with their source files.

## Key Invariants

- **INV-6**: Agent surface cannot modify Canonical Knowledge. Admin tools are not registered on agent surface.
- **P-1**: All context compilation is deterministic. Same input + same knowledge_version = same output.
- **P-3**: All Canonical mutations require human approval (Observation → Proposed → Canonical).

## Tag Mappings Layer (Outside Canonical DAG)

- **Storage**: `tag_mappings` table — separate from Canonical Knowledge, no approval workflow
- **CRUD**: Direct repository methods (`upsertTagMapping`, `setTagMappings`, `getDocumentsByTags`, etc.)
- **Approved filter**: `getDocumentsByTags` JOIN-filters on `documents.status = 'approved'`
- **Source**: `slm` (small language model) or `manual` (human-curated)
- **IntentTagger port**: `extractTags(plan) → IntentTag[]` (async, FakeTagger for tests, OllamaIntentTagger for production)
- **Wired to ContextCompiler**: `plan` + tagger → expanded context via tag_mappings lookup

## Adapter Meta (Outside Canonical Knowledge)

- **Storage**: `adapter_meta` table — separate from Canonical Knowledge, no approval workflow
- **Same pattern as**: `tag_mappings` (operational metadata, direct CRUD)
- **Purpose**: Track the package version that last ran a full `deploy-adapters`
- **Written by**: `deploy-adapters` CLI only (full deploy, no `--targets`, no failures)
- **Read by**: MCP server startup (to set `adapterOutdated` flag for `notices`)

## Project Structure

```
src/
  core/
    store/      — SQLite repository, schema, database
    read/       — ContextCompiler (deterministic DAG routing)
    init/       — Stack detection, template loading, bootstrap, upgrade
    automation/ — ObservationAnalyzer port, RuleBasedAnalyzer, ReviewCorrectionAnalyzer,
                  PrMergedAnalyzer, ManualNoteAnalyzer, ProposeService
    tagging/    — IntentTagger port (tag extraction interface)
    types.ts    — All TypeScript type definitions
  mcp/
    server.ts   — MCP tool registration (surface-conditional)
    services.ts — Service facade (INV-6 enforcement)
  adapters/
    cursor/     — .cursor/rules/ rule generation
    Codex/     — AGENTS.md section injection
    types.ts    — Adapter shared types
  expansion/
    ollama-client.ts — Ollama REST API client
    intent-tagger.ts — OllamaIntentTagger (IntentTagger implementation)
  e2e/          — End-to-end integration tests
  main.ts       — Entry point (--surface, --db, --templates, --ollama-*)
templates/      — Reserved for custom user templates (--template-dir)
```

## Design Decisions

- Agent surface registers 6 tools (compile_context, observe, get_compile_audit, get_known_tags, workspace_status, init_detect)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fuwasegu/aegis](https://github.com/fuwasegu/aegis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
