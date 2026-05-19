---
trigger: always_on
description: > **Purpose:** Ground orientation for any agentic coding assistant working on the LQ.AI codebase. Read this first; it points at the right reference for any decision and lays out the project's standards in one place.
---

# Orientation for Claude Code (and other coding assistants)

> **Purpose:** Ground orientation for any agentic coding assistant working on the LQ.AI codebase. Read this first; it points at the right reference for any decision and lays out the project's standards in one place.
>
> **Audience:** Claude Code, Cursor, Aider, or any human or agent making implementation decisions. Read in full before the first contribution; refer back as needed.

---

## What this project is

LQ.AI is an open-source AI platform for in-house legal teams. Self-hosted; bring-your-own-keys; runs in the operator's environment. Skills are open-source work product, not closed prompts. The Inference Gateway is the security boundary — the only component holding privileged provider API keys.

The project's reason for existing — and its central design constraint — is **transparency**. Every artifact that shapes the user experience is visible work product. A skill that produces a wrong answer should be readable, debuggable, and forkable by the user who relies on it. This is not a marketing principle; it is an architectural commitment that affects every implementation decision.

Read [README.md](README.md) for the public-facing description. Read [docs/PRD.md §1.3 Transparency as a Founding Principle](docs/PRD.md#13-transparency-as-a-founding-principle) for the full philosophical grounding.

---

## Decision routing

When you face a decision while implementing, the canonical reference is — in priority order:

1. **The PRD** ([docs/PRD.md](docs/PRD.md)) — for product, capability, and architectural decisions.
2. **The OpenAPI sketches** ([docs/api/backend-openapi.yaml](docs/api/backend-openapi.yaml), [docs/api/gateway-openapi.yaml](docs/api/gateway-openapi.yaml)) — for endpoint shapes, request/response schemas, status codes.
3. **The database schema** ([docs/db-schema.md](docs/db-schema.md)) — for tables, columns, indexes, constraints.
4. **The gateway configuration example** ([gateway.yaml.example](gateway.yaml.example)) — for the gateway's configuration shape.
5. **The implementation order** ([docs/M1-IMPLEMENTATION-ORDER.md](docs/M1-IMPLEMENTATION-ORDER.md)) — for which task is next and what its acceptance criteria are.
6. **The skill-authoring guide** ([docs/skill-authoring-guide.md](docs/skill-authoring-guide.md)) — for skill conventions.
7. **CONTRIBUTING.md** ([CONTRIBUTING.md](CONTRIBUTING.md)) — for code style, testing, PR process.

If a decision is not anchored in any of these, **stop and ask** rather than guess. The right move is usually:

1. Document the decision in the appropriate place (PRD §9 if it's forward-looking; an ADR in `docs/adr/` if it's structural; CLAUDE.md if it's a workflow convention).
2. Resume implementation.

This is more friction than letting an agent decide independently, and that friction is worth it. The cost of an undocumented decision compounds across every subsequent task that relates to it.

---

## What the codebase looks like

```
lq-ai/
│
├── api/                # FastAPI backend service (Python)
├── gateway/            # Inference Gateway service (Python)
├── web/                # OpenWebUI fork (TypeScript/JavaScript)
├── word-addin/         # Office.js Word add-in (M3)
├── tests/              # Cross-cutting integration tests
├── scripts/            # Operational scripts (backfills, one-time tools)
│
├── skills/             # The 10 starter skills (filesystem-canonical)
│   ├── nda-review/
│   ├── msa-review-saas/
│   └── ...
│
├── docs/               # PRD, architecture, contribution guides, schemas
├── deploy/             # Helm chart, deployment recipes
└── docker-compose.yml
```

Each subsystem (`api/`, `gateway/`, `web/`) is a self-contained service. They talk over HTTP using OpenAPI-defined contracts. There is no shared in-process code — adapters cross the boundary explicitly.

---

## Code style

### Python (`api/`, `gateway/`, `scripts/`)

- **Formatter:** `ruff format` (Black-compatible).
- **Linter:** `ruff check` with the project's `ruff.toml`.
- **Type checker:** `mypy` strict mode for `gateway/`; standard mode for `api/`.
- **Type annotations:** required on all public functions and class methods.
- **Async:** prefer `async def` for I/O-bound code; use `httpx.AsyncClient`, not `requests`.
- **Exceptions:** use the `lq_ai.errors` exception hierarchy; do not raise bare `Exception`.

### JavaScript / TypeScript (`web/`)

- **Formatter:** Prettier per `.prettierrc`.
- **Linter:** ESLint per `.eslintrc`.
- **TypeScript:** required for new files; legacy `.js` files migrate gradually.
- **Framework:** SvelteKit. The `web/` subdirectory is a fork of OpenWebUI, which is a SvelteKit app — extensions and customizations stay in Svelte. We do **not** mix React into `web/`. The Word add-in (`word-addin/`, M3) uses Office.js with React; the `web/` codebase does not.
- **Component conventions:** match the OpenWebUI conventions for shared components; use the project's design system primitives rather than ad-hoc Tailwind.

### Both

- **Imperative-mood commit messages** ("Add X" not "Added X").

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LegalQuants/lq-ai](https://github.com/LegalQuants/lq-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
