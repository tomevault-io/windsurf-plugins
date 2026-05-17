---
trigger: always_on
description: Multi-tenant SCIM 2.0 Service Provider (RFC 7643/7644) with five Maven modules:
---

# SCIM 2.0 Playground - Copilot Instructions

## Architecture

Multi-tenant SCIM 2.0 Service Provider (RFC 7643/7644) with five Maven modules:

| Module | Role | Port |
|---|---|---|
| `scim-server-common` | Shared JPA entities, repositories, and common security utilities | - |
| `scim-server-api` | SCIM 2.0 API (Users, Groups, Discovery, Bulk) | 8080 |
| `scim-server-mgmt` | Admin UI + management API (Thymeleaf + vanilla JS) | 8081 |
| `scim-validator` | Groovy/Spock SCIM compliance suite (REST Assured) | - |
| `scim-validator-mgmt` | Validator run/inspection management service | 8082 |

Multi-tenancy is workspace-based. SCIM routes are scoped to `/ws/{workspaceId}/scim/v2/**`,
and the current implementation expects `workspaceId` to be a UUID.

- `BearerTokenAuthFilter` extracts the workspace UUID from the path and validates that the token belongs to that workspace.
- Bearer tokens are validated via SHA-256 hash lookup (`WorkspaceTokenRepository.findByTokenHashAndNotRevoked`).
- There is no `WorkspaceContext` ThreadLocal anymore; controllers resolve the workspace UUID from the route and pass it explicitly into services.
- All core SCIM entities are workspace-scoped with `workspace_id` foreign keys.

Compatibility mode is route-based and extensible:

- Controllers expose both base and compat route forms, for example:
	- `/ws/{workspaceId}/scim/v2/Users`
	- `/ws/{workspaceId}/scim/v2/{compat}/Users`
- `CompatMode` currently supports `MS`.
- `MsScimUserMapper` applies Microsoft validator compatibility tweaks:
	- converts selected `primary` booleans to string values
	- adds flattened enterprise manager alias key

Management security uses Auth0 OIDC:

- Both management modules use standard Spring Security OAuth2 Client with Auth0 as the OIDC provider.
- Each module has its own `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`, and `AUTH0_ISSUER_URI`.
- Role claims are read from a configurable OIDC claim (`APP_SECURITY_OIDC_ROLE_CLAIM`, default `https://scimplayground.dev/roles`).
- Management user persistence is email-based in both management modules; resolved emails are normalized and stored as the primary key.
- Management access expects a usable email claim from OIDC principals.
- Shared helpers live in `scim-server-common` (`Auth0OidcSecuritySupport`, `MgmtSecuritySupport`).

Kubernetes support is split into two trees:

- `k8s/app/**` deploys the namespaced SCIM stack in `scim`:
	- CloudNativePG PostgreSQL cluster
	- validator database resource
	- API, management, and validator-mgmt Deployments and Services
- `k8s/cluster/**` deploys supporting cluster resources:
	- local-path storage configuration and `local-path-custom` `StorageClass`
	- `cloudflared` in its own namespace

Kubernetes secrets are stored as `*.sops.yaml` files and rendered through `ksops`.
The root `.sops.yaml` defines the active age recipient.

## Build And Run

```bash
# Full reactor build
mvn clean install

# Full reactor build without running validator specs
mvn clean install -Dskip.validator.tests=true

# API local mode (requires datasource env vars and ACTUATOR_API_KEY)
cd scim-server-api && mvn spring-boot:run

# Mgmt UI/API local mode (requires datasource env vars, ACTUATOR_API_KEY, and
# Auth0 OIDC env vars: AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_ISSUER_URI,
# AUTH0_REDIRECT_URI — or set SPRING_PROFILES_ACTIVE=cloudflare for Cloudflare JWT mode)
cd scim-server-mgmt && mvn spring-boot:run

# Validator management local mode (requires datasource env vars, ACTUATOR_API_KEY, and
# Auth0 OIDC env vars: AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_ISSUER_URI,
# AUTH0_REDIRECT_URI — or set SPRING_PROFILES_ACTIVE=cloudflare for Cloudflare JWT mode)
cd scim-validator-mgmt && mvn spring-boot:run

# Docker stack
docker compose up --build

# Docker stack plus local cloudflared sidecar
docker compose --profile cloudflare up --build

# Kubernetes support resources (requires kubectl, kustomize, ksops, sops, and SOPS_AGE_KEY_FILE)
kustomize build --enable-alpha-plugins --enable-exec k8s/cluster | kubectl apply -f -

# Kubernetes application stack
kustomize build --enable-alpha-plugins --enable-exec k8s/app | kubectl apply -f -
```

Docker default ports:

- API `:8080`
- Mgmt `:8081`
- Validator Mgmt `:8082`
- Playground PostgreSQL `:5432`
- Validator PostgreSQL `:5433`

Operational notes:

- `docker-compose.yml` loads `docker/env/cloudflare.env` into the management apps.
- Kubernetes manifests set `SPRING_PROFILES_ACTIVE=cloudflare` for the management apps.
- Application services in Kubernetes are `ClusterIP`; Cloudflare tunnel is the external-access path in this branch.
- No repository-specific `DOCKER_HOST` or `TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE` overrides are required for local Testcontainers runs; use the default local Docker Desktop / Docker Engine setup.

## Validator Execution

`scim-validator` can either bootstrap its own disposable target via
Testcontainers or run against an already reachable SCIM API.

```bash
cd scim-validator && mvn test
```

Notes from `ScimBaseSpec`:

- By default, the validator can bootstrap PostgreSQL plus `edipal/scim-server-api:latest` when explicit `SCIM_*` settings are not provided.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edipal/scim](https://github.com/edipal/scim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
