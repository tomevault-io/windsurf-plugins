---
trigger: always_on
description: > **Note for AI Assistants**: This is the root context file providing project overview. For detailed implementation context:
---

# CLAUDE.md - LiteMaaS AI Context File

> **Note for AI Assistants**: This is the root context file providing project overview. For detailed implementation context:
>
> - **Backend Context**: [`backend/CLAUDE.md`](backend/CLAUDE.md) - Fastify API implementation details
> - **Frontend Context**: [`frontend/CLAUDE.md`](frontend/CLAUDE.md) - React/PatternFly 6 implementation details
> - **Project Structure**: [`docs/architecture/project-structure.md`](docs/architecture/project-structure.md) - Complete directory structure
> - **Changelog**: [`CHANGELOG.md`](CHANGELOG.md) - All notable changes per release (Keep a Changelog format)
> - **Documentation**: See `docs/` for comprehensive guides

## 🚀 Project Overview

**LiteMaaS** is a model subscription and management platform that bridges users and AI model services through LiteLLM integration.

**Monorepo** with two packages:

- **Backend** (`@litemaas/backend`): Fastify API server with PostgreSQL, OAuth2/OIDC/JWT, RBAC
- **Frontend** (`@litemaas/frontend`): React + PatternFly 6 UI with 9-language i18n support

**Tech Stack**: Fastify, React, TypeScript, PostgreSQL, PatternFly 6, LiteLLM integration

## 📁 Project Structure

See [`docs/architecture/project-structure.md`](docs/architecture/project-structure.md) for complete directory structure and file organization.

## 🔧 Key Features

**Role-Based Access Control (RBAC)**: Three-tier hierarchy `admin > adminReadonly > user` with OpenShift integration.

**Model Capability Management**: Multi-type model support beyond chat-only models:

- **Model Types**: Chat (default), Embeddings, Document Conversion — selected via radio group in admin model form
- **Tokenize Capability**: Optional per-model toggle for tokenization support
- **Document Conversion**: Docling provider integration with `/health` endpoint testing, hidden irrelevant fields (backend model name, TPM, costs, max tokens)
- **Capability Labels**: Color-coded flair labels on model cards (Chat=blue, Embeddings=green, Tokenize=orangered, Document Conversion=orange)
- **Type-Specific Curl Examples**: View Key modal shows contextual curl commands based on model type
- **Chat Playground Filtering**: Only chat-capable models shown in playground
- **Redis Cache Flush**: Optional Redis integration (`REDIS_HOST`/`REDIS_PORT`) to flush LiteLLM's cache after model CRUD, ensuring all proxy pods pick up changes immediately
- **Deployment**: Redis deployment included in Helm (`redis.enabled: true`) and Kustomize charts

**Restricted Model Subscription Approval** (Major feature - 2025 Q4): Admin-controlled access to sensitive/costly models with comprehensive approval workflow:

- **Restricted Model Flagging**: Administrators mark models requiring approval
- **Three-state workflow**: Pending → Active/Denied with request review capability
- **Bulk Operations**: Approve/deny multiple requests with detailed result tracking
- **Full Audit Trail**: Complete history in `subscription_status_history` table
- **Granular RBAC**: Read/write/delete permissions (admin vs adminReadonly)
- **Automatic Cascade**: Access revocation when models become restricted
- **LiteLLM-first security**: API key updates prioritize access revocation

**Admin Usage Analytics** (Major feature - 2025 Q3): Enterprise-grade analytics with comprehensive system-wide visibility:

- **Day-by-day incremental caching** with intelligent TTL (permanent historical, 5-min current day)
- **Multi-dimensional filtering**: users, models, providers, API keys with cascading filter dependencies
- **Trend analysis** with automatic comparison period calculations
- **Rich visualizations**: usage trends, model distribution, weekly heatmap (component ready, integration pending)
- **Data export**: CSV/JSON with filter preservation
- **Configurable cache TTL** via ConfigContext integration with React Query

**Admin User Management** (Major feature - 2025 Q4): Consolidated admin interface for managing users through a modal-based workflow with tabbed views:

- **Unified Management Modal**: Profile, Budget & Limits, API Keys, and Subscriptions tabs
- **Role Management**: Admin/adminReadonly/user role toggles with conflict detection
- **Budget & Rate Limits**: Max budget, budget duration, TPM, and RPM with real-time spend from LiteLLM, spend reset, and color-coded utilization progress bars
- **API Key Lifecycle**: Create, view, edit quotas (including per-model limits and expiration), soft revoke, permanent delete, and spend reset
- **Subscription Management**: Add/remove model subscriptions directly from user modal with automatic LiteLLM key sync
- **Full Audit Trail**: All admin actions logged with metadata
- **RBAC**: `users:read` (admin, adminReadonly) for viewing, `users:write` (admin only) for modifications

**Admin Audit Log**: Full audit log viewer at `/admin/audit` with category/action filtering, human-readable labels, API access toggle, search, and date range filters. Requires `admin:audit` permission (admin + adminReadonly).

**API Key Quota Management**: Comprehensive budget and rate limit management for API keys across user self-service and admin interfaces:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rh-aiservices-bu/litemaas](https://github.com/rh-aiservices-bu/litemaas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
