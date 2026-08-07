---
trigger: always_on
description: > **This is the agent context file for Legalise.** Coding agents pick it up
---

# Legalise — Agent Context (AGENTS.md)

> **This is the agent context file for Legalise.** Coding agents pick it up
> automatically when working in the repo; you can also drop it into any AI agent
> or paste it into a chat to talk about Legalise — what it is, how it's built,
> and what it does and does not claim.
> It is a self-contained snapshot synthesised from the project's own docs
> (`README.md`, `docs/ARCHITECTURE.md`, `docs/TRUST.md`, `docs/THREAT_MODEL.md`).
> The house rule throughout, the same one the codebase holds itself to: a
> capability is only described as live if the code implements it. Anything
> deferred, dormant, or unmounted is named as such.
>
> Repo: https://github.com/b1rdmania/legalise · Licence: Apache 2.0 · Status:
> open-source **evaluation release**, not for live client matters.

---

## 1. The one-paragraph version

Legalise is an open-source **governance layer for legal AI**: human sign-off
plus a tamper-evident audit trail for AI-assisted legal work, built for England
& Wales solicitor practice. It runs locally, you bring your own model key, and
it is an evaluation release — not a regulated legal service. The whole system
exists to make one loop legible:

> **draft → cite → sign-off → audit**

AI prepares an output inside a *matter*, cites the documents it used, a named
solicitor reviews and signs it (the signature pins the exact output by hash),
and every step writes to an audit log the application cannot edit or delete.
**AI is preparation, not the deliverable.** The audit trail makes the work
inspectable; the signature makes a human accountable. The thesis is deliberately
narrow: *the machine signs its own record; the human signs the work* — and the
two are kept separate everywhere.

It's an early-stage, mostly solo project shared in the open — a working
exploration of how this *could* be done, not a finished or proven product. Treat
the claims below as "this is what the code does today", not "this is solved".

## 2. What it's built to answer (the four questions)

A regulator, insurer, supervisor, or partner eventually asks four questions
about any AI tool used on a matter. Legalise is an attempt to make them
answerable from the record rather than from trust:

1. **What did it see?** Every matter has a spine — documents, chronology,
   parties, retention clock, privilege posture. The AI sees only what lives in
   the matter; cross-matter access is scoped in the application layer and every
   access is audited.
2. **Under what protection?** Every matter carries a *privilege posture*, read
   from the database before each model call (`A_cleared` / `B_mixed` /
   `C_paused`).
3. **What did it produce?** Prompt, response, model, tokens, latency, posture,
   and calling module are hashed and stored. Any interaction reconstructs from
   the audit row.
4. **Who stayed accountable?** A named human signs each output, and the record
   shows whether the signer was the author. Every model call, mutation,
   chronology entry, and denial writes one append-only audit row, mirrored into
   a hash chain.

**One caveat, stated plainly:** this is tamper-*evidence*, not tamper-*proofing*.
A database superuser can still rewrite and re-link history. External anchoring
would close that and is not built.

---

## 3. Stack (boring by design)

- **Backend:** FastAPI, async throughout. SQLAlchemy 2 (async) + Alembic.
- **Database:** PostgreSQL 16 + pgvector — one store for relational data, JSONB,
  full-text, and embeddings.
- **Auth:** `fastapi-users`, cookie transport (HttpOnly/Secure/SameSite=Lax),
  DB-backed access tokens (real revocation), email verification via Resend.
- **Frontend:** React 19 + Vite + Tailwind. TanStack Router, **path-based**
  (legacy `#/…` hash URLs are rewritten to path URLs on boot).
- **Document conversion / extraction:** Gotenberg (HTML→PDF), LibreOffice
  headless (DOCX), `pypdf` / `pdfplumber` / `python-docx`.
- **PII detection:** Microsoft Presidio + spaCy.
- **Local dev stack:** Postgres, MinIO, Redis, Gotenberg, FastAPI, React via
  docker-compose.
- **Hosting (single eval instance):** Cloudflare in front, Fly.io (lhr) backend,
  Neon (London) Postgres, Cloudflare R2 (EU) for blobs.

---

## 4. The core concepts

### 4.1 The matter is the unit of everything

A "matter" is the primitive. The unit of **isolation, authorisation, and audit**
is always one matter. Every matter-scoped route checks ownership
(`Matter.created_by_id == user.id`) — one user cannot read another's matter.
Capability grants, audit scope, privilege posture, and the audit hash chain are
all keyed per matter. The matter model carries slug, parties, matter type,
status, `privilege_posture`, `default_model_id`, and a JSONB facts blob.

The workspace is **matter-first and chat-led**: the assistant chat is the
primary surface, with documents, skills, activity, and approvals summoned as tabs
around it. The worked sample matter is **Khan v Acme**.

### 4.2 The audit substrate (the load-bearing part)

Two layers: a plain audit log, and a synchronous hash chain over it.

- **Audit entries.** Every consequential action writes a row to `audit_entries`.
  Read endpoints deliberately emit nothing. Three write paths:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b1rdmania/legalise](https://github.com/b1rdmania/legalise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
