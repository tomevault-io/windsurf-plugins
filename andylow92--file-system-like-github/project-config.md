---
trigger: always_on
description: > **Entry point for AI agents working in this repository.**
---

# AGENTS.md

> **Entry point for AI agents working in this repository.**
> Read this file first. It tells you where the knowledge lives and the rules
> you must follow while working here.

This project is a **local-first markdown workspace** (a GitHub-style file tree
over a filesystem-backed API) that is being grown into a **shared "brain"**:
plain markdown files that **humans edit and read**, and that **agents can also
read, search, link, and write** — with every change legible to the human.

If you are an agent operating this vault, you do not have to scrape the UI:
there is a first-class **MCP server** (`apps/mcp`) that exposes the vault as
tools, and every write you make is recorded with attribution so the human can
see it. See _What's built_ below.

---

## Where the knowledge lives (route yourself here)

Always orient yourself using these documents before acting:

| You want to know...                              | Read this                                                        |
| ------------------------------------------------ | ---------------------------------------------------------------- |
| What exists, what's done, what's next            | [`docs/implementation.md`](docs/implementation.md)               |
| How to run, test, and validate the app           | [`docs/implementation.md`](docs/implementation.md) → _Commands_  |
| Manual integration checks                        | [`docs/integration-test-plan.md`](docs/integration-test-plan.md) |
| Backend API endpoints + request/response shapes  | [`apps/api/README.md`](apps/api/README.md)                       |
| Agent tools (MCP) + how writes are attributed    | [`apps/mcp/README.md`](apps/mcp/README.md)                       |
| Connect an MCP host (OpenClaw / Claude / Cursor) | [`docs/CONNECT.md`](docs/CONNECT.md)                             |
| Project overview / human-facing pitch            | [`README.md`](README.md)                                         |

`docs/implementation.md` is the **source of truth for project state**. When you
finish a unit of work, update it (see _Goal-driven execution_ below).

---

## Agent quick-reference (intent → MCP tool → HTTP endpoint)

The MCP server (`apps/mcp`) is your surface; each tool maps 1:1 to an endpoint on
the in-process API. Pick the tool that matches your intent:

| I want to…                                             | MCP tool            | HTTP endpoint                |
| ------------------------------------------------------ | ------------------- | ---------------------------- |
| List notes (optionally under a subtree)                | `list_notes`        | `GET /api/tree`              |
| Read a note (by `path` **or** `id`)                    | `read_note`         | `GET /api/file`              |
| Read one `^block` (+ surrounding context)              | `read_block`        | `GET /api/block`             |
| List a note's `^block` anchors                         | `get_block_anchors` | `GET /api/block-anchors`     |
| Create a note                                          | `create_note`       | `POST /api/file`             |
| Overwrite a note (pass `etag`)                         | `update_note`       | `PUT /api/file`              |
| Surgical edit (append/replace section/block, `dryRun`) | `patch_note`        | `PATCH /api/file`            |
| Full-text / `#tag` search                              | `search_notes`      | `GET /api/search`            |
| Semantic (TF-IDF) search                               | `semantic_search`   | `GET /api/semantic-search`   |
| Hybrid (RRF) search                                    | `hybrid_search`     | `GET /api/hybrid-search`     |
| RAG context bundle (matches + focus-note neighbors)    | `get_context`       | `GET /api/context`           |
| Cited answer kit + offline gap analysis                | `think`             | `GET /api/think`             |
| Backlinks (incl. `rel:` type)                          | `get_backlinks`     | `GET /api/backlinks`         |
| Whole vault wikilink graph                             | `get_graph`         | `GET /api/graph`             |
| Recent provenance / audit trail                        | `recent_activity`   | `GET /api/audit`             |
| Question log + recurring knowledge gaps                | `recent_questions`  | `GET /api/questions`         |
| Create a folder                                        | `create_folder`     | `POST /api/dir`              |
| Move / rename a note or folder                         | `move_path`         | `PATCH /api/path`            |
| Delete a note or folder                                | `delete_path`       | `DELETE /api/path`           |
| Propose a create/update/delete for human review        | `propose_edit`      | `POST /api/proposals`        |
| List proposals + their status                          | `list_proposals`    | `GET /api/proposals`         |
| List skill notes (procedural playbooks)                | `list_skills`       | `GET /api/skills`            |
| Run the dream-cycle maintenance scan                   | `run_maintenance`   | `POST /api/maintenance/scan` |
| Learn from reviewed draft→final outreach pairs         | `run_feedback`      | `POST /api/feedback/scan`    |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andylow92/file-system-like-github](https://github.com/andylow92/file-system-like-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
