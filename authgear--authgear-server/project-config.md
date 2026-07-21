---
trigger: always_on
description: This repository is Authgear Server. Use the repo docs and skills below as the source of truth for agentic work.
---

# AGENTS.md

This repository is Authgear Server. Use the repo docs and skills below as the source of truth for agentic work.

## Repository layout

```
authgear-server/
├── cmd/                        # Entry points (main packages)
│   ├── authgear/               # Core auth server binary
│   │   ├── main.go
│   │   ├── server/             # HTTP server wiring
│   │   ├── background/         # Background worker (cron jobs, queues)
│   │   ├── adminapi/           # Admin API server
│   │   ├── config/             # Config commands (validate, migrate)
│   │   └── images/             # Docker image utilities
│   ├── portal/                 # Portal backend binary
│   │   └── main.go
│   └── once/                   # One-shot migration / setup commands
│
├── pkg/                        # Shared Go packages
│   ├── lib/                    # Core business logic (auth flows, OAuth,
│   │   │                         sessions, SAML, OIDC, webhooks, etc.)
│   ├── auth/                   # AuthUI HTTP handlers & webapp routes
│   │   ├── handler/            # Request handlers
│   │   └── webapp/             # Server-side rendered web-app views
│   ├── admin/                  # Admin API GraphQL layer
│   │   └── graphql/
│   ├── portal/                 # Portal backend GraphQL layer
│   │   └── graphql/
│   ├── siteadmin/              # Site-admin API (global administration)
│   ├── api/                    # Shared API types / transport helpers
│   ├── resolver/               # Request resolver middleware
│   ├── util/                   # General-purpose utilities
│   └── images/                 # Image-processing helpers
│
├── authui/                     # AuthUI frontend (React/TypeScript)
│   │                             Compiled output is embedded into the binary.
│   └── src/
│
├── portal/                     # Portal frontend (React/TypeScript)
│   │                             The portal lets tenant admins manage their apps.
│   └── src/
│
├── resources/                  # Static resources embedded into binaries
│   ├── authgear/               # AuthUI templates, translations, assets
│   └── portal/                 # Portal static assets
│
├── e2e/                        # End-to-end test suite
│   └── tests/
│
├── docs/                       # Specs and design documents
│   ├── specs/                  # Feature and API specifications
│   └── api/                    # OpenAPI / schema definitions
│
├── hack/                       # Developer scripts and tooling
├── devtools/                   # Local development helpers
└── scripts/                    # CI / release scripts
```

**Key mappings:**

| What you want to change | Where to look |
|---|---|
| Auth UI (login/signup pages) | `authui/src/` (frontend), `pkg/auth/webapp/` (handlers), `resources/authgear/` (templates) |
| Portal (admin console UI) | `portal/src/` (frontend), `pkg/portal/graphql/` (backend) |
| Admin API (GraphQL) | `pkg/admin/graphql/`, `cmd/authgear/adminapi/` |
| Site Admin API | `pkg/siteadmin/`, `cmd/authgear/` |
| Core auth logic (OAuth, OIDC, sessions, flows) | `pkg/lib/` |
| Background jobs / workers | `cmd/authgear/background/` |
| Config schema & validation | `pkg/lib/config/`, `cmd/authgear/config/` |
| E2E tests | `e2e/tests/` |

## Documentation map

| Doc | Contents |
|---|---|
| [README.md](README.md) | Project overview, local setup, build, running Authgear |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution workflow, coding standards, testing, commit/PR process |
| [docs/specs/convention.md](docs/specs/convention.md) | Spec writing convention — required reading before authoring a new spec |
| [docs/specs/api.md](docs/specs/api.md) | Authgear public API spec (OAuth/OIDC, flows, endpoints) |
| [docs/specs/api-admin.md](docs/specs/api-admin.md) | Admin API spec (GraphQL, endpoints, auth) |
| [docs/specs/](docs/specs/) | Feature-specific specs — authoritative source for behavior rules |
| [docs/api/](docs/api/) | OpenAPI / schema definitions |
| [portal/docs/ARCHITECTURE.md](portal/docs/ARCHITECTURE.md) | Portal architecture: stack, GraphQL endpoints, providers, config/theming |
| [portal/docs/FRONTEND.md](portal/docs/FRONTEND.md) | Portal React SPA conventions: routing, GraphQL, styling, i18n, forms |
| [portal/docs/storybook.md](portal/docs/storybook.md) | Storybook conventions — read before adding or editing component stories |

## Common commands

Day-to-day shortcuts. See [CONTRIBUTING.md](CONTRIBUTING.md) for full setup.

### Start local dev

When the user says **"start local development environment"**, run these in order, each in its own terminal (commands 2–4 are long-running):

1. `docker compose up -d` — bring up Postgres, Redis, MinIO, etc.
2. `make start` — main auth server.
3. `make start-portal` — portal backend.
4. `cd portal && npm start` — portal frontend (Vite dev server).

Add `make authui-dev` as a fifth terminal only when editing AuthUI. Assumes the env is already set up per CONTRIBUTING.md.

### Other commands

| Command | What it does |
|---|---|
| `make vendor` | One-time bootstrap after cloning: installs golangci-lint, fetches Go deps, and builds the frontends. |
| `make test` | Run Go tests under `cmd/...` and `pkg/...`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [authgear/authgear-server](https://github.com/authgear/authgear-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
