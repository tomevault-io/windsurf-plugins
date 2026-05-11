---
trigger: always_on
description: **[[Flywheel]] Memory** — MCP tools that search, write, and auto-link your Obsidian vault — and learn from your edits. 21 merged action-param tools across 3 preset tiers (agent/power/full) organized into 12 categories: search, read, write, graph, schema, wikilinks, corrections, tasks, memory, note-ops, temporal, and diagnostics — all local, all markdown. Hybrid search (BM25 + semantic via Reciprocal Rank Fusion) is available when embeddings are built via `init_semantic`.
---

# Flywheel Memory - Claude Code Instructions

**[[Flywheel]] Memory** — MCP tools that search, write, and auto-link your Obsidian vault — and learn from your edits. 21 merged action-param tools across 3 preset tiers (agent/power/full) organized into 12 categories: search, read, write, graph, schema, wikilinks, corrections, tasks, memory, note-ops, temporal, and diagnostics — all local, all markdown. Hybrid search (BM25 + semantic via Reciprocal Rank Fusion) is available when embeddings are built via `init_semantic`.

---

## Git Workflow

**No direct pushes to main.** Branch protection is enforced — all changes require a PR.

- Create a feature branch for code changes
- Push the branch and open a PR via `gh pr create`
- Docs-only changes still need a PR but can be fast-tracked
- Never run `npm run build` while a benchmark is running

---

## Architecture

### Source Structure

```
packages/mcp-server/src/
├── index.ts                    # MCP server entry point + tool preset gating
├── tool-registry.ts            # Tool gating, tiering, activation tracking
├── config.ts                   # Tool categories, tiers, presets, instructions
├── tools/
│   ├── toolCatalog.ts          # Tool metadata collection for embedding manifest
│   ├── read/                   # Read-side tool registrations (20 files, helpers omitted)
│   │   ├── query.ts            # search
│   │   ├── primitives.ts       # read (structure|section|sections), tasks (list|toggle)
│   │   ├── graphAnalysis.ts    # graph (analyse|backlinks|forward_links|strong_connections|path|neighbors|strength|cooccurrence_gaps)
│   │   ├── system.ts           # refresh_index, entity (list|alias|suggest_aliases|merge|suggest_merges|dismiss_merge)
│   │   ├── health.ts           # doctor (health|pipeline|config|log|stats)
│   │   ├── schemaTools.ts      # schema (overview|field_values|conventions|folders|rename_field|rename_tag|migrate|validate|note_intelligence)
│   │   ├── noteIntelligence.ts # insights (evolution|staleness|context|note_intelligence|growth)
│   │   ├── wikilinks.ts        # link (suggest|validate|feedback|stubs|unlinked|dashboard|unsuppress|timeline|...)
│   │   ├── migrations.ts       # (schema actions: rename_field, migrate absorbed into schema)
│   │   ├── metrics.ts          # (insights action: growth)
│   │   ├── semantic.ts         # init_semantic
│   │   └── brief.ts            # (memory action: brief)
│   └── write/                  # Write-side tool registrations
│       ├── mutations.ts        # edit_section (add|remove|replace)
│       ├── tasks.ts            # vault_add_task (standalone), tasks(action: toggle) in primitives.ts
│       ├── notes.ts            # note (create|delete) and legacy note files
│       ├── move-notes.ts       # note (move|rename)
│       ├── frontmatter.ts      # vault_update_frontmatter
│       ├── entity.ts           # entity (alias|merge) + correct (record|list|resolve|undo)
│       ├── corrections.ts      # correct tool source logic
│       ├── wikilinkFeedback.ts # link (feedback action)
│       ├── tags.ts             # schema (rename_tag action)
│       ├── memory.ts           # memory (store|get|search|list|forget|summarize_session|brief)
│       ├── config.ts           # (doctor action: config)
│       ├── system.ts           # (correct action: undo)
│       └── policy.ts           # policy
├── core/
│   ├── read/                   # Read-side core logic (graph, vault, parser, fts5, config, watcher)
│   │   └── toolRouting.ts      # Semantic tool routing, manifest loading
│   ├── write/                  # Write-side core logic (writer, wikilinks, git, validator, policy engine)
│   ├── shared/                 # Shared utilities (recency, cooccurrence, retrievalCooccurrence, hub export, stemmer, metrics, indexActivity, toolTracking, graphSnapshots, toolSelectionFeedback)
│   └── semantic/               # Semantic search (embeddings.ts — embedding generation, similarity.ts — hybrid ranking)
└── generated/
    └── tool-embeddings.generated.ts  # Pre-computed tool embedding manifest
```

### Multi-Vault & Transport

```
packages/mcp-server/src/
├── vault-registry.ts              # VaultContext interface + VaultRegistry class + parseVaultConfig()
├── index.ts                       # applyToolGating(), registerAllTools(), createConfiguredServer()
```

- `vault-registry.ts` — `VaultContext` holds per-vault state (name, vaultPath, stateDb, vaultIndex, flywheelConfig, watcher). `VaultRegistry` tracks all contexts with a primary vault name. `parseVaultConfig()` reads `FLYWHEEL_VAULTS` env var.
- `applyToolGating()` — Monkey-patches `server.tool()` to filter by category. In multi-vault mode, wraps handlers with `activateVault()` and injects optional `vault` parameter on all tools.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velvetmonkey/flywheel-memory](https://github.com/velvetmonkey/flywheel-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
