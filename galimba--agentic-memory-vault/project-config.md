---
trigger: always_on
description: > This file is loaded at the start of every agent session. It is the single source of truth for how agents interact with this vault. Co-evolve this file with your team as conventions stabilize.
---

# CLAUDE.md — Vault Agent Configuration

> This file is loaded at the start of every agent session. It is the single source of truth for how agents interact with this vault. Co-evolve this file with your team as conventions stabilize.

## Identity

- **Vault Name**: `{{VAULT_NAME}}` <!-- Replace during initialization -->
- **Organization**: `{{ORG_NAME}}` <!-- Replace during initialization -->
- **Vault Version**: `0.4.0`
- **Initialized**: `{{INIT_DATE}}` <!-- Replace during initialization -->
- **Primary Agent Platform**: `{{PLATFORM}}` <!-- claude-code | codex | copilot | cursor | custom -->

## Architecture

This vault follows the **Karpathy LLM Wiki** pattern with extensions for multi-agent enterprise use.

### Three-Layer Structure

```
raw/          # Layer 1: Immutable source documents. NEVER modify files here.
wiki/         # Layer 2: Agent-generated knowledge pages. Agents own this layer entirely.
memory/       # Layer 3: Operational state — decisions, logs, running notes.
```

### Support Directories

```
.vault/       # Vault configuration, rules, schemas, hooks, scripts. Human-managed.
templates/    # Frontmatter and content templates for each document type.
docs/         # Vault documentation for humans. How-to guides, onboarding.
```

### Critical Files

| File | Location | Purpose |
|------|----------|---------|
| `CLAUDE.md` | Root | Agent configuration (this file) |
| `AGENTS.md` | Root | Platform-agnostic agent instructions (mirrors this file) |
| `CODEX.md` | Root | OpenAI Codex-specific overrides |
| `index.md` | `wiki/` | Master catalog of all wiki pages |
| `log.md` | `wiki/` | Append-only chronological record of all operations |
| `status.md` | `memory/` | Current vault health and operational state |

## Operations

Agents perform exactly **three operations** on this vault:

### 1. INGEST — Process new source material

```
Trigger: New file appears in raw/ OR human requests ingestion
Steps:
  1. Read the source document in raw/
  2. Create or update a summary page in wiki/sources/
  3. Update wiki/index.md with the new entry
  4. Update every materially affected wiki page (concepts, entities, comparisons)
  5. Append an entry to wiki/log.md
  6. Validate all modified files against .vault/schemas/
  7. Verify tags comply with .vault/rules/tags.md
```

### 2. QUERY — Answer questions using the vault

```
Trigger: Human or agent asks a question
Steps:
  1. Read wiki/index.md to locate relevant pages
  2. Read those pages and synthesize an answer
  3. Cite sources using [[wikilinks]]
  4. Optionally file the answer back as a new wiki page
  5. Append query record to wiki/log.md
```

### 3. LINT — Health check the vault

```
Trigger: Scheduled or manual request
Steps:
  1. Check for contradictions between wiki pages
  2. Find orphan pages (no inbound links)
  3. Identify stale content against per-domain / per-type thresholds
     in .vault/schemas/staleness-config.json
  4. Verify all pages have valid frontmatter
  5. Verify all tags are from the approved taxonomy
  6. Check compliance with .vault/rules/
  7. Write findings to memory/notes/lint-report-YYYY-MM-DD.md
     (run `vault-tools.sh lint --report`; also emitted by `doctor`)
  8. Suggest new pages, connections, or questions
```

## Rules

### Hard Rules (Enforced — violations block commits)

All hard rules are defined in `.vault/rules/hard-rules.md`. Summary:

1. **NEVER modify files in `raw/`**. This directory is immutable.
2. **Every file in `wiki/` MUST have valid YAML frontmatter** per `.vault/schemas/frontmatter.md`.
3. **Every file in `wiki/` MUST include at least one tag** from the approved taxonomy.
4. **Markdown files should stay under 200 lines** (warning) and **MUST NOT exceed 400 lines** (hard limit). Split into linked sub-pages if needed.
5. **Code files should stay under 400 lines** (warning) and **MUST NOT exceed 600 lines** (hard limit). Modularize by splitting into focused files with a single entry point.
6. **All wiki page titles MUST be unique** across the vault.
7. **Frontmatter `updated` field MUST reflect the actual last-modified date**.
8. **No file may exist in `wiki/` without a corresponding entry in `wiki/index.md`**.
9. **Tags MUST use flat prefix notation**: `domain/engineering`, not nested hierarchies.
10. **Binary files (images, PDFs) MUST be stored in `raw/`**, never in `wiki/` or `memory/`.
11. **No agent may modify `.vault/rules/`, `.vault/hooks/`, or `.vault/scripts/`**. Governance changes require human PRs.
12. **No agent may modify `CLAUDE.md`, `AGENTS.md`, or `CODEX.md`**. Agent instruction changes require human PRs.
13. **No agent may modify `.github/` or `templates/`**. CI and template changes require human PRs.
14. **Do not delete files from `wiki/` or `memory/`.** Set `status: archived` in frontmatter instead. Use `VAULT_ALLOW_DELETE=1` for cleanup.
15. **Log files (`wiki/log.md`, `memory/logs/`) are append-only**. Deletions are rejected by HR-015. Set `LOG_EDIT_ALLOWED=1` to bypass for legitimate corrections.

### Soft Rules (Configurable — adapt to your workflow)

All soft rules are defined in `.vault/rules/soft-rules.md`. Defaults:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galimba/agentic-memory-vault](https://github.com/galimba/agentic-memory-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
