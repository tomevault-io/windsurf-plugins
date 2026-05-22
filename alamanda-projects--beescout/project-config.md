---
trigger: always_on
description: > This file is the **technical guardrail for AI agents (and developers)** working on this codebase.
---

# BeeScout — Claude Code Guardrails

> This file is the **technical guardrail for AI agents (and developers)** working on this codebase.
> It contains conventions, gotchas, and rules-of-thumb that prevent common mistakes.
> It does **not** explain what BeeScout is or who it's for — see the docs below for that.

## Where to find the rest

| Looking for... | Read this |
|---|---|
| What BeeScout does, stack, repo layout | [README.md](README.md) |
| Local setup & environment variables | [getting-started.md](getting-started.md), [.env.example](.env.example) |
| Deploy ke production + checklist go-live | [docs/deploy-production.md](docs/deploy-production.md) |
| Who BeeScout is built for (4 personas) | [docs/personas.md](docs/personas.md) |
| Business ↔ technical term mapping | [docs/glossary.md](docs/glossary.md) |
| How to contribute (incl. AI usage policy) | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Who decides what, how to become a maintainer | [GOVERNANCE.md](GOVERNANCE.md) |
| Reporting vulnerabilities | [SECURITY.md](SECURITY.md) |
| Canonical data contract schema | [data-contract/examples/full.yaml](data-contract/examples/full.yaml) |
| Approval workflow, rule catalog, YAML import | [docs/](docs/) |

When you need user-facing personas (Pak Bambang/Bu Retno/Mas Dimas/Mbak Indah), reference `docs/personas.md` rather than re-deriving them.

---

## Working Mode

The maintainer (single, [@haninp](https://github.com/haninp)) operates **brainstorm-heavy** — most input is "ide / fitur / bug report" in Bahasa Indonesia. **Your job is to execute**: investigate the codebase, propose a concrete plan when the task is non-trivial, implement, run QA, and open a PR.

- **Language**: Maintainer writes in Bahasa Indonesia. Reply in the same language. User-facing UI strings are Indonesian (follow [docs/glossary.md](docs/glossary.md)); code identifiers, file names, commit subjects, and PR titles stay in English (conventional-commit style).
- **PR is the unit of delivery** — every change goes through a PR, even one-line fixes. Maintainer reviews & merges from mobile (GitHub mobile app).
- **Issues are the source of truth** for pending work — not chat history, not memory. If a non-trivial decision emerges mid-task, leave a comment on the relevant issue/PR before the chat moves on. Future agents (cloud or local) read issues, not transcripts.
- **`docs/sdlc.md`** is the canonical lifecycle. Skip steps only when the change is truly trivial (typo, comment fix).

### Definition of Done (before opening a PR)

1. **Tests pass locally**: `make test` (backend + both frontend typechecks).
2. **QA scripts pass**: every `scripts/qa-*.sh` relevant to the change exits 0. Two run on every PR via CI — keep them green: `scripts/qa-form-buttons.sh` (form button safety) and `scripts/qa-prod-readiness.sh` (production-readiness static checks; `.env`-dependent checks auto-skip when no `.env` is present).
3. **New convention discovered? Document it.** Add a section to this file (and a longer write-up under `docs/` if it deserves one). Future you / future agent will need it.
4. **PR description** includes: short summary, "Closes #N" trailer, test plan checklist.
5. **Branch naming**: `<type>/<issue#>-<slug>` where `type` ∈ `fix | feat | chore | docs | refactor`. Example: `fix/12-unique-contract-number`.
6. **Squash-merge only** (project default). The branch is deleted on merge.

### When the task is non-trivial

Use the plan mode (Claude Code) or write a plan file to `.github/` / comment on the issue **before** writing code. A plan should include: context (why), scope (what), files to touch, risk/trade-offs, and verification approach. Mirror the style of existing plans under `.claude/plans/` if available, or follow [docs/sdlc.md](docs/sdlc.md)'s "Design" step.

---

## Backend Conventions

- **All routes in `repository/app/main.py`** — single-file by design (small surface area). Don't introduce a router split without a [Tech Proposal](.github/ISSUE_TEMPLATE/tech-proposal.yml).
- **Pydantic models in `repository/app/model/`** — always check these before adding fields to the frontend form. Frontend types in `frontend-admin/src/types/` (and mirrored in `frontend-user/`) must stay in sync.
- **`retention`** is stored as `int` + separate `retention_unit: str` (values: `tahun | bulan | pekan | hari | jam`). Never combine into one string.
- **JWT tokens** live in httpOnly cookies — never localStorage. Default access token expiry: 180 minutes (3 hours).
- **MongoDB collections** are configured via env (`MONGODB_COL_*`). Common ones: `dgr` (contracts), `dgrusr` (users), `approvals`, `domains` (standardised team domains — see below). See `core/connection.py`.
- **`data_domain` is an access key** — it must match contract `metadata.consumer[].name` via exact string. The `domains` collection (managed via `/domain/*`, admin only) is the curated source of valid slugs. New user create/edit validates `data_domain` against it through `validate_data_domain()` in `main.py` — but validation is **skipped while the `domains` collection is empty** (backward compatible). Slugs are lowercase, hyphenated (`slugify_domain()`); domains are soft-deleted (`is_active: false`), never hard-deleted.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alamanda-projects/beescout](https://github.com/alamanda-projects/beescout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
