---
trigger: always_on
description: This repository is the instruction layer for a clean greenfield rebuild of the VinUni Career Platform.
---

# Claude Code Operating Guide — VinUni Career Platform

This repository is the instruction layer for a clean greenfield rebuild of the VinUni Career Platform.

## Prime Directive

- `backend/` and `frontend/` may be absent after reset. If absent, scaffold them from `CLAUDE.md`, `.claude/`, and `docs/`; if present, treat them as the current implementation.
- If `backend/` or `frontend/` contain only `.env` / `.env.example`, treat them as env seeds, not implemented code.
- Root `.env` / `.env.example` are reserved for AI hook logging. Backend runtime env belongs in `backend/.env`; frontend runtime env belongs in `frontend/.env`.
- Do not reuse pre-reset backend/frontend code, old git history, or remembered Phase 0/1/2 status as foundation.
- Root legacy config files may still exist during reset. Do not treat root `docker-compose.yml`, `pyrightconfig.json`, old Makefiles, or old README content as source of truth until regenerated for the new scaffold.
- Docs are the source of truth for product intent and business rules. When code conflicts with docs, reconcile by fixing code OR flagging the doc as stale — do not silently discard working code.
- For large work: explore relevant docs/files -> plan -> implement -> test -> summarize.
- Do not deploy, push production changes, change production env values, or remove sponsored/ad labels.

## Immediate Post-Claude Review Gate — 2026-07-04

The latest code review found that the current implementation is not yet a
release-ready product even though many surfaces now exist. Agents must clear
these blockers before broad feature expansion or any "complete" status update:

- Backend quality gates are not green (re-verified 2026-07-08: `ruff` 120
  errors — 21 in `app`, rest cosmetic in `tests`; `mypy app` 45 errors / 15
  files; `pytest` 1936 passed / 6 failed). `SPONSORED_SURFACES` is now defined
  (that blocker is resolved). Remaining real blockers: onboarding imports a
  non-existent `async_session_factory` in `doc_verification.py` (runtime crash),
  nullable auth principal passed where non-null `UUID` is expected across the
  onboarding router, salary/experience presenter/service type gaps in
  `opportunities`, `Result.rowcount` misuse in moderation services, and one
  module-boundary violation (documents/platform_admin reaching into other
  modules' `domain.models`).
- Frontend typecheck/build pass, but build warnings remain in public jobs and
  notifications hook dependencies. Treat these as regression risks for heavily
  used surfaces.
- Auth/register must be email/password only. Name/profile fields belong to
  onboarding or profile/CV confirmation. Verification/reset emails must not
  assume `name` exists, and vi/en user-facing copy must be i18n-backed.
- Workflow builder must persist real canvas changes, use selectors for users,
  departments, stages, templates, prompts, and actions, expose validation/dry
  run/history, and avoid raw IDs as the operator-facing configuration model.
- CV Studio has a useful canvas start, but template versioning/assets,
  renderer/PDF/preview unification, browser QA, accessibility, and university
  template governance remain open.
- Product status must distinguish `implemented`, `API wired`,
  `browser verified`, `visual-design verified`, and `E2E verified`. Older
  historical "clean" status lines do not override the latest gate results.

When in doubt, run a blocker stabilization pass first and update
`docs/IMPLEMENTATION_STATUS.md` with exact commands and pass/fail evidence.

## Owner Decisions — 2026-07-10

These override any earlier doc/status lines that conflict with them:

- **Applications are always identified.** Anonymous apply, blind-screening, and
  the identity-reveal handshake are removed product-wide. Do NOT reintroduce a
  `candidate_identity`/reveal capability, anonymized candidate cards, or a
  redacted CV preview. This removed only the anonymity dance — partner CV access
  stays RBAC-gated per user/role/department (`candidate_access` capability), CV
  downloads stay watermarked, and application-open/CV-view/CV-download stay
  audited. See `docs/SECURITY_PRIVACY.md`, `docs/PARTNER_RBAC_ANALYTICS_SPEC.md`,
  `docs/BUSINESS_LOGIC.md` §4/§8. (Note: anonymous **company reviews** and the
  privacy-safe **guest discovery session** are separate features and stay.)
- **Talent Pool is AI semantic search.** pgvector embeddings over consented
  candidate CVs + skill/experience filters + LLM rerank that returns
  human-readable match reasons (never a raw similarity score). It must support
  **external-JD search**: paste/upload a JD that is not yet a posted job and find
  matching candidates. No provider/model/token/embedding internals are ever
  exposed to partners; deterministic keyword+filter fallback when AI is down.
  Contract in `docs/PARTNER_RBAC_ANALYTICS_SPEC.md`.
- **Advertising is a real allocation engine**, not "upload a banner":
  auto-allocation/distribution of paid placements into defined sponsored slots,
  partner targeting by coarse LOCATION and by student MAJOR/CAREER (never exact
  GPS or sensitive categories), strict separation of organic vs recommended vs
  sponsored vs university-curated inventory, and truthful non-removable paid

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielngo03/TER-Career](https://github.com/danielngo03/TER-Career) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
