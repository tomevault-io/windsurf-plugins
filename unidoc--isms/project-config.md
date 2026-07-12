---
trigger: always_on
description: A versioned document management platform. Documents live in git (markdown + YAML frontmatter). Collaboration (reviews, comments, approvals, tasks) lives in PostgreSQL. Web UI (Vue) for readers and management. CLI for managers.
---

# ISMS Platform — Claude Code Guide

## What is this?

A versioned document management platform. Documents live in git (markdown + YAML frontmatter). Collaboration (reviews, comments, approvals, tasks) lives in PostgreSQL. Web UI (Vue) for readers and management. CLI for managers.

The core is a **generic document engine** — it knows nothing about ISO 27001, clauses, controls, or any specific standard. All standard-specific content comes from **templates** that are loaded from disk. Templates provide the starting document structure; after scaffolding, everything is just documents.

## Architecture

### Core = Document Engine

The core provides:
- **Git-backed documents** — markdown files with YAML frontmatter, versioned
- **Review workflow** — send for review, inline comments, approve/reject, merge
- **Registers** — risks, assets, suppliers, systems, legal requirements, incidents (all in PostgreSQL)
- **Multi-tenant** — organizations with RLS, per-org identifiers, white-label branding
- **Authentication** — OIDC, password+TOTP, passkeys, API tokens

The core does NOT know about:
- Specific standards (ISO 27001, SOC 2, etc.)
- Folder names (clauses, controls, policies — those are template choices)
- SoA generation (that's a template-level document)
- Document types — everything is a "document" with a `document_id`

### Templates = Standard-Specific Content

Templates live on disk at `ISMS_TEMPLATE_PATH` (separate git repo). Each template is a directory of markdown files that get scaffolded into an org's git repo. Templates provide:
- Folder structure (template maintainer chooses names)
- Document content (markdown with TODO placeholders)
- `.title` files for folder display names
- `meta.yaml` with template identity and version

After scaffolding, the org owns the content. Templates are not referenced again — the org's git repo is the source of truth.

### Data Split

```
Git repo (per org)          PostgreSQL (shared)           Blob store (per org)
├── README.md               ├── users & organizations     {org-uuid}/
├── documents/              ├── reviews, comments         ├── branding/
│   ├── <template>/         ├── approvals, decision log   │   ├── logo.svg
│   │   ├── <folders>/      ├── tasks, changes            │   └── favicon.ico
│   │   │   ├── .title      ├── suggestions               └── evidence/
│   │   │   └── *.md        ├── risks, assets, suppliers      └── {checkin-id}/
│   │   └── ...             ├── systems, incidents
│   └── <other-template>/   ├── corrective actions        ISMS_STORAGE_BACKEND=file → disk
                            ├── audit programmes, findings ISMS_STORAGE_BACKEND=s3 → S3/R2
                            ├── objectives, programs
                            ├── legal requirements
                            ├── notifications, activity
                            ├── entity changelog
                            ├── approval policies
                            └── entity references
```

Git stores ONLY documents (markdown + frontmatter). No images, no branding, no binary files. Branding and evidence go to the blob store (`internal/isms/blob/`).

## CRITICAL RULES

### Document versioning model
Three layers of history, each with a distinct purpose:
- **Git commits** = raw edit history. Every save is a commit. This is the working log.
- **`document_versions`** = official milestones only. Created on merge/publish and confirm. NOT on draft edits. This is what the Version History UI shows.
- **`decision_log`** = governance trail. Approvals, rejections, merges, confirmations with content hashes.

Version numbers increment when an approved document is edited into a new draft (status goes from `approved` → `draft`). They do NOT increment on every save. Draft edits are working state, not milestones.

### Review status transitions
Review status is a state machine. Only dedicated endpoints may transition between states:
- `open` → `approved` (via approve endpoint)
- `open` → `changes_requested` (via approve endpoint with changes_requested/proposed_revision)
- `changes_requested` → `open` (via resubmit)
- `approved` → `merged` (via merge endpoint)
- Any active status → `closed` (via status endpoint — the ONLY transition it allows)

The `PUT /reviews/:id/status` endpoint accepts ONLY `closed`. All other transitions go through their dedicated handlers. Never bypass this.

### NEVER use git CLI on the server
All git operations on the server MUST use go-git library. The ONLY exception is `api_git.go` for the wire protocol.

### Everything is a document
There are no "clauses", "policies", or "controls" in the core. All documents have `document_id` in frontmatter. Use `store.FindDocumentByID()` to resolve paths. Never hardcode folder names.

### Templates from disk, not embedded
Templates are loaded from `ISMS_TEMPLATE_PATH`. Use `scaffold.ListTemplates()` and `scaffold.IsValidTemplate()`. Never embed template content in the binary.

### Document IDs are lowercase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unidoc/isms](https://github.com/unidoc/isms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
