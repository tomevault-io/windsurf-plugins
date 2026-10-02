---
trigger: always_on
description: Ignitify is a self-hosted deployment and operations control plane built with Rust and Vue 3. It persists state in SQLite and runs a local worker that reconciles deployments. The current product includes:
---

# Repository Guidelines

## Project Overview

Ignitify is a self-hosted deployment and operations control plane built with Rust and Vue 3. It persists state in SQLite and runs a local worker that reconciles deployments. The current product includes:

- password bootstrap/login, JWT access tokens, rotating hashed refresh sessions, step-up authentication, audit context, and role-gated operator controls;
- projects, encrypted project and service environment variables, image, Compose, and Git-backed services;
- queued deployment, rollback, cancel, stop, deployment events, and server-sent deployment logs;
- local Docker, restricted Compose, and SSH remote-server runtimes; Git source builds using Dockerfile, static, Railpack, or reviewed Compose sources;
- Traefik ingress, managed application and control-plane routes, certificates, DNS verification, domain policy, infrastructure settings, and ingress fallback configuration;
- GitHub/GitLab/Gitea providers, provider tests and repository/branch discovery, remote BuildKit builders, runtime container inspection/actions, controlled terminals, host metrics, uptime monitoring, and remote-agent heartbeats;
- operator-managed notification channels for deployment and backup events via Telegram, Discord, SMTP, Resend, or custom HTTPS webhooks;
- an operator-configured OpenAI-compatible AI assistant with encrypted API-key storage, a global floating chat surface, and bounded diagnostic context from deployment, container, and terminal logs;
- offline SQLite/runtime-secret backup and restore, with optional S3-compatible upload, scheduled S3 runs, and backup-run history.

The product executes real Docker, Compose, SSH, DNS, HTTP-monitoring, Git, and S3 effects when configured. Treat all runtime and infrastructure code as security-sensitive. Never claim a capability is only a UI fixture unless the relevant implementation is actually absent.

## Architecture And Data Flow

- The Rust workspace uses Rust 2024 and shared dependencies from root `Cargo.toml`.
- `ignitify-core` is the binary composition root. It loads runtime secrets/configuration, dispatches `backup` and `restore` CLI operations, builds adapters and workers, binds the loopback listener, and calls `axum::serve`. It owns no HTTP routes, handlers, request/response DTOs, or `IntoResponse` mapping.
- `ignitify-api` owns Axum route registration, handlers, HTTP DTOs, request extractors, cookies/origin checks, WebSocket/SSE adapters, static frontend serving, OpenAPI/Swagger documentation, audit context, and safe API-error mapping. It owns the OpenAI-compatible AI configuration and chat HTTP adapter. A handler authenticates, authorizes, validates, calls a service/repository, records audit context where required, then maps the result.
- `ignitify-auth` owns Argon2 credentials, bootstrap and step-up flow, JWT access tokens, rotating hashed refresh-token families, session DTOs, and `AuthError`. It receives `Database` through `AuthService::new(database, config)`.
- `ignitify-db` owns SQLite connection setup, embedded migrations, persistence records, and repositories. It is the authoritative state store for users, projects, services, deployments, domains, settings, providers, AI configuration, remote infrastructure, monitoring, notification channels, audit activity, and backup destinations.
- `ignitify-domain` owns transport-agnostic validation and domain types. It must not import SQLx, Axum, authentication, Docker, or runtime types.
- `ignitify-control-plane` owns service configuration encryption/read models, deployment submission, worker reconciliation, stream publication, and runtime/ingress/source-build traits. HTTP submits or reads state; the worker and adapters own external effects and retries.
- Runtime/infrastructure adapters implement the control-plane contracts: `ignitify-runtime-docker`, `ignitify-runtime-compose`, `ignitify-runtime-remote`, `ignitify-ingress-traefik`, `ignitify-source-git`, and `ignitify-dns`. `ignitify-monitoring` runs the uptime worker; `ignitify-notifications` dispatches deployment and backup events; `ignitify-terminal` owns PTY primitives; `ignitify-backup-s3` owns S3 upload signing and transport.
- Keep dependencies acyclic. The usual flow is `core -> api -> auth/control-plane/db/domain and adapters`; `control-plane -> db/domain`; adapters depend on control-plane/domain/db only where their contract requires it. Lower layers must never import `ignitify-api` or `ignitify-core`.
- Frontend flow: `src/main.ts` installs Pinia, i18n, and Router; the router initializes `useAuthStore`; API calls attach the memory-only Bearer token; refresh uses a strict HttpOnly cookie; Vite proxies `/api` in development. Production requests are served from the embedded frontend bundle by the backend.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xFlawlessDev/ignitify](https://github.com/xFlawlessDev/ignitify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
