---
trigger: always_on
description: **SkillHub** is an **enterprise-grade, self-hosted agent skill registry** for publishing,
---

# SkillHub — AGENTS.md

**SkillHub** is an **enterprise-grade, self-hosted agent skill registry** for publishing,
discovering, and managing reusable skill packages across an organization. It provides a **REST API
backend**, a **React web UI**, a **security scanner**, and a **ClawHub CLI compatibility layer**.

## Quick Reference

| Item       | Value                                                      |
|------------|------------------------------------------------------------|
| Backend    | Spring Boot 3.2.3, Java 21, Maven multi-module (7 modules) |
| Frontend   | React 19, TypeScript, Vite, pnpm                           |
| Scanner    | Python (FastAPI), port 8000                                |
| Database   | PostgreSQL 16 (Flyway migrations)                          |
| Cache      | Redis 7 (sessions, distributed locks, idempotency)         |
| Storage    | LocalFile (dev) / S3/MinIO (prod)                          |
| Build      | `make dev-all` (dev), `make staging` (pre-PR)              |
| Docs       | `docs/` (design), `document/` (VitePress user guide)       |
| CI         | GitHub Actions (`.github/workflows/`)                      |

## Directory Map

```
skillhub/
├── server/                          # Maven multi-module Spring Boot backend
│   ├── skillhub-app/                # Application layer: bootstrap, controllers, assembly
│   │   ├── bootstrap/               # Bootstrap admin & local dev data initializers
│   │   ├── compat/                  # ClawHub CLI compatibility layer controllers
│   │   ├── config/                  # Spring configuration classes
│   │   ├── controller/              # REST controllers (transport only)
│   │   │   ├── admin/               # Admin controllers (user mgmt, labels, search)
│   │   │   ├── portal/              # Portal controllers (skills, governance, security)
│   │   │   └── support/             # Package extractors (zip, multipart)
│   │   ├── dto/                     # Request/response DTOs
│   │   ├── exception/               # Exception handling
│   │   ├── filter/                  # Servlet filters (auth context, rate limiting)
│   │   ├── listener/                # Event listeners (notification recipients, etc.)
│   │   ├── metrics/                 # Micrometer metrics
│   │   ├── projection/              # Lifecycle projection models
│   │   ├── ratelimit/               # Rate limiting logic
│   │   ├── repository/              # Query repositories (read-model assembly)
│   │   ├── security/                # Security configuration
│   │   ├── service/                 # App services (workflow orchestration)
│   │   ├── stream/                  # SSE streaming endpoints
│   │   ├── task/                    # Background task scheduling
│   │   └── SkillhubApplication.java # Spring Boot entry point
│   │
│   ├── skillhub-domain/             # Domain layer: entities, rules, services (innermost)
│   │   ├── audit/                   # AuditLog entity, repository, service
│   │   ├── auth/                    # Password reset entities
│   │   ├── event/                   # Domain event classes (SkillPublishedEvent, etc.)
│   │   ├── governance/              # Governance notification service
│   │   ├── idempotency/             # Idempotency records
│   │   ├── label/                   # Skill label management
│   │   ├── namespace/               # Namespace, members, roles, policies
│   │   ├── report/                  # Skill reporting/governance
│   │   ├── review/                  # Review tasks, promotion requests
│   │   ├── security/                # Security scanning domain model
│   │   ├── shared/                  # Shared domain utilities
│   │   │   └── exception/           # Domain exceptions (LocalizedDomainException, etc.)
│   │   ├── skill/                   # Core skill entities and services
│   │   │   ├── metadata/            # SKILL.md frontmatter parsing
│   │   │   ├── service/             # Skill domain services (publish, query, governance)
│   │   │   └── validation/          # Package validation (SkillPackagePolicy, etc.)
│   │   ├── social/                  # Star, rating, subscription entities
│   │   └── user/                    # UserAccount, profile moderation
│   │
│   ├── skillhub-auth/               # Authentication & authorization
│   │   ├── config/                  # Spring Security configuration
│   │   ├── device/                  # OAuth Device Flow for CLI auth
│   │   ├── identity/                # Identity binding service
│   │   ├── local/                   # Local (password) auth
│   │   ├── merge/                   # Account merging
│   │   ├── oauth/                   # OAuth2 login handlers
│   │   ├── policy/                  # Route security policies
│   │   ├── rbac/                    # RBAC service and role definitions
│   │   ├── token/                   # API token management
│   │   └── user/                    # User-related auth services
│   │
│   ├── skillhub-search/             # Search SPI + PostgreSQL full-text implementation
│   │   ├── postgres/                # PostgresFullTextIndexService, QueryService
│   │   └── service/                 # Search SPI interfaces
│   │
│   ├── skillhub-storage/            # Object storage SPI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iflytek/skillhub](https://github.com/iflytek/skillhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
