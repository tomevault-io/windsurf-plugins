---
trigger: always_on
description: Before writing any code, always discuss the problem first. Ask clarifying questions until the requirements, constraints, and approach are clear. Only start implementing once the user has confirmed the plan. If a task is ambiguous, propose options and let the user choose rather than making assumptions.
---

# Velane — Codex Agent Guide

## Working style

Before writing any code, always discuss the problem first. Ask clarifying questions until the requirements, constraints, and approach are clear. Only start implementing once the user has confirmed the plan. If a task is ambiguous, propose options and let the user choose rather than making assumptions.

## Repo layout

```
velane/
├── services/
│   ├── control-plane/      # Go 1.26 API server (chi, pgx/v5, zap)
│   ├── executor-runtime/   # Bun + Python sandboxed HTTP runners
│   ├── mcp-server/         # MCP protocol server for Cursor / Claude Code
│   └── cli/                # Cobra CLI (velane login / push / invoke)
├── apps/
│   ├── admin/              # Vite + React admin portal
│   └── embed-dashboard/    # Vite + React embeddable viewer
└── platform-libraries/     # Canonical source for built-in libs (bun/ + python/)
```

Each Go service has its own `go.mod`. Module paths:
- `github.com/abskrj/velane/services/control-plane`
- `github.com/abskrj/velane/services/mcp-server`
- `github.com/abskrj/velane/services/cli`

## Essential commands

```bash
make up           # docker compose up --build -d
make down         # docker compose down -v
make build        # copy-platform-libs + go build ./...
cd services/control-plane && go test ./...
cd apps/admin && npx tsc --noEmit
```

## Go conventions (control-plane)

- **Never commit** unless explicitly asked.
- Always run `go build ./...` from inside `services/control-plane/` — not from the repo root.
- **Tenant isolation** — every slug-based handler must verify the authenticated tenant matches the URL slug. Pattern:
  ```go
  authTenant := middleware.TenantFromContext(r.Context())
  if authTenant == nil || authTenant.ID != tenant.ID {
      writeError(w, http.StatusForbidden, "access denied")
      return
  }
  ```
- **Scope middleware** — all authenticated routes must use `middleware.RequireScope(scope, log)`. Minimum scopes: `invoke` for reads, `manage` for writes, `admin` for destructive/team actions.
- **Error helpers** — use `writeError(w, status, msg)` and `writeJSON(w, status, v)` from `handlers/helpers.go`.
- **API key prefix** is `vl_`. Embed token prefix is `et_`. Do not change these.
- Migrations live in `internal/store/postgres/migrations/`. Number them sequentially (`011_`, `012_`, …).
- Platform libraries are embedded via `//go:embed all:files` in `internal/platformlibs/loader.go`. Always run `make copy-platform-libs` before building.

## Auth model

| Credential | Where stored | Validated by |
|---|---|---|
| Session JWT (RS256) | `localStorage.sessionToken` | `SessionAuth` middleware → `JWTProvider.ValidateSession` |
| API key (`vl_…`) | `localStorage.apiKey` | `Auth` middleware → `ValidateAPIKey` |
| Embed token (`et_…`) | `localStorage.apiKey` | `AuthEmbed` middleware or `Auth` middleware (synthetic key) |

JWT access tokens expire in 15 min; refresh tokens last 7 days. `ValidateSession` checks issuer — do not remove that check.

Embed tokens get a synthetic API key with scopes `["invoke", "manage"]` — they must NOT have `admin` scope.

## Frontend conventions (apps/admin)

- API calls go through `src/lib/api.ts`. The `request()` function handles 401s automatically.
- Tenant slug comes from `localStorage.getItem('tenantSlug')`.
- Tailwind only — no CSS files, no inline `style=`. Monochrome palette (`gray-900` primary, `gray-50`/`gray-100` backgrounds).
- Primary buttons: `rounded-lg bg-gray-900 text-white hover:bg-gray-800`.
- Do not add `console.log` statements.
- Type-check before reporting done: `npx tsc --noEmit`.

## Libraries design

Platform libraries are grouped by **integration** (e.g. Salesforce, Google Sheets, Google Docs). Each integration can have multiple capability slugs (e.g. `salesforce-cases`, `salesforce-contacts`). The Libraries UI shows integrations as the top level and lists their capabilities beneath each one.

**Platform library code must export a class**, not standalone functions:

```typescript
// Bun — index.ts
export class SalesforceCases {
  constructor(private config?: { instanceUrl?: string; accessToken?: string }) {}
  async createCase(fields: CaseFields): Promise<CreateCaseResult> { … }
  async getCase(id: string): Promise<CaseRecord> { … }
  async updateCase(id: string, fields: Partial<CaseFields>): Promise<void> { … }
  async deleteCase(id: string): Promise<void> { … }
}
export default SalesforceCases
```

```python
# Python — module.py
class SalesforceCases:
    def __init__(self, instance_url=None, access_token=None): …
    def create_case(self, fields: dict) -> dict: …
    def get_case(self, case_id: str) -> dict: …
```

The `meta.json` must include an `integration` field:
```json
{ "name": "Cases", "integration": "Salesforce", "description": "…" }
```

## Adding a new platform library

1. Create `platform-libraries/{bun|python}/<slug>/` with:
   - `index.ts` (Bun) or `module.py` (Python) — export a class as default
   - `meta.json` — `{"name": "…", "integration": "…", "description": "…"}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abskrj/velane](https://github.com/abskrj/velane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
