---
trigger: always_on
description: This file provides context for AI agents working on this codebase.
---

# AGENTS.md

This file provides context for AI agents working on this codebase.

## Project Overview

dmarc-msp is a Python tool and Docker-based deployment stack that automates DMARC monitoring across multiple client domains for Managed Service Providers (MSPs). It integrates with [parsedmarc](https://github.com/domainaware/parsedmarc) as the underlying report processor.

One email address receives all DMARC reports. parsedmarc routes them to per-client OpenSearch indices via a YAML domain-to-index-prefix mapping. dmarc-msp manages the lifecycle: DNS authorization records, OpenSearch multi-tenancy, dashboard provisioning, and retention policies.

## Architecture Principles

- **Service layer is king** — all business logic lives in `dmarc_msp/services/`. The CLI and API are thin wrappers.
- **CLI-first, server-optional** — the CLI calls services directly. The API server is optional.
- **Plugin-based DNS** — DNS providers implement `DNSProvider` (see `dns_providers/base.py`).
- **Idempotent operations** — onboarding the same domain twice is safe.
- **Normalize early** — all domain names and index prefixes are lowercased at the point of entry.
- **DB before external state** — `add_domain` commits the domain reservation to the database before creating DNS records or modifying the parsedmarc YAML. This prevents concurrent operations (e.g., `cleanup-dns`) from seeing orphaned external state.
- **YAML rollback on failure** — every orchestrator that mutates the parsedmarc YAML (`add_domain`, `remove_domain`, `move_domain`, `offboard_client`) restores the YAML to its prior state if the DB commit or a later step fails.

## Key Directories

```text
dmarc_msp/
├── config.py           # Pydantic Settings — YAML + env + Docker secrets
├── models.py           # Pydantic models (API responses, inter-service data)
├── db.py               # SQLAlchemy ORM (Client, Domain, AuditLog) + SQLite
├── services/           # Core business logic
│   ├── clients.py      # Client CRUD + rename
│   ├── dns.py          # DMARC authorization record lifecycle (RFC 7489)
│   ├── opensearch.py   # Tenant/role/role-mapping provisioning
│   ├── dashboards.py   # NDJSON rewrite + saved object import
│   ├── parsedmarc.py   # YAML domain mapping management
│   ├── retention.py    # ISM policy management + email cleanup
│   ├── onboarding.py   # Orchestrator: add/remove/move domains
│   └── offboarding.py  # Orchestrator: full client teardown
├── dns_providers/      # Pluggable DNS backends
│   ├── base.py         # Abstract DNSProvider + DNSRecord (get/create/delete/list)
│   ├── cloudflare.py   # Cloudflare (default, included in base deps)
│   ├── route53.py      # AWS Route 53 (optional extra)
│   ├── gcp.py          # Google Cloud DNS (optional extra)
│   └── azure.py        # Azure DNS (optional extra)
├── process/
│   └── docker.py       # Send SIGHUP to parsedmarc container
├── cli/                # Typer CLI (dmarcmsp)
│   ├── helpers.py      # Dependency wiring for CLI commands
│   ├── client.py       # client create/list/show/update/rename/offboard
│   ├── domain.py       # domain add/remove/move/verify/list/cleanup-dns/bulk-*
│   ├── tenant.py       # tenant provision/deprovision
│   ├── dashboard.py    # dashboard import / import-all
│   ├── retention.py    # retention cleanup-emails/ensure-default-policy
│   ├── parsedmarc.py   # parsedmarc reload
│   └── server.py       # serve command (API server)
└── api/                # FastAPI management API
    ├── dependencies.py # DI (settings, db session, services)
    ├── middleware.py    # IP allowlist
    ├── schemas.py      # Request/response models
    └── routers/        # One router per resource

deploy/
├── postfix/            # Custom receive-only Postfix container
│   ├── Dockerfile      # Alpine + Postfix
│   ├── main.cf.template # Receive-only config (envsubst'd)
│   ├── master.cf       # Port 25 + 587 listeners
│   └── entrypoint.sh   # TLS detection, envsubst, Maildir setup
├── nginx/              # TLS-terminating reverse proxy
│   ├── Dockerfile      # nginx:alpine
│   ├── nginx.conf.template              # Full HTTPS config
│   ├── nginx-http-only.conf.template    # Bootstrap config (no certs yet)
│   ├── mta-sts.conf.template            # MTA-STS HTTPS server block
│   ├── mta-sts-http-only.conf.template  # MTA-STS bootstrap (ACME only)
│   └── entrypoint.sh   # Cert detection, MTA-STS DNS check, auto-reload
├── certbot/            # Let's Encrypt config
│   ├── cli.ini         # Certbot defaults (webroot, agree-tos)
│   └── entrypoint.sh   # Cert requests (main + conditional MTA-STS), renewal loop
├── opensearch/         # OpenSearch node config
│   └── opensearch.yml
└── dashboards/         # Dashboards config
    └── opensearch_dashboards.yml
```

## Docker Services

| Service | Image | Purpose |
| --- | -- |- -- |
| `postfix` | Custom (Alpine + Postfix) | Receive-only SMTP for DMARC reports |
| `parsedmarc` | `ghcr.io/domainaware/parsedmarc:latest` | Report processing |
| `opensearch` | `opensearchproject/opensearch:3` | Data storage (backend only) |
| `opensearch-dashboards` | `opensearchproject/opensearch-dashboards:3` | Visualization (internal, behind nginx) |
| `nginx` | Custom (nginx:alpine) | TLS termination, reverse proxy, rate limiting |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [domainaware/dmarc-msp](https://github.com/domainaware/dmarc-msp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
