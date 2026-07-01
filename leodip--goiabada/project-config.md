---
trigger: always_on
description: Open-source authentication server in Go. OAuth2/OIDC compliant with SSO, 2FA, and admin console.
---

# Goiabada - OAuth2/OIDC Authentication Server

## What It Is
Open-source authentication server in Go. OAuth2/OIDC compliant with SSO, 2FA, and admin console.

## Architecture (3 Go Modules)

```
src/
├── core/           # Shared: models, data layer, oauth, validators
├── authserver/     # OAuth2/OIDC endpoints, user auth flows
└── adminconsole/   # Admin UI for managing users/clients/permissions
```

- **Core** (`src/core/go.mod`): Database interface, models, JWT handling, OAuth logic
- **Auth Server** (`src/authserver/go.mod`): Main auth endpoints, token issuance
- **Admin Console** (`src/adminconsole/go.mod`): Admin management UI

## Key Directories

### Core (`src/core/`)
- `models/` - All domain models (Client, User, Permission, Group, etc.)
- `data/` - Database interface + implementations (commondb/, mysqldb/, postgresdb/, sqlitedb/, mssqldb/)
- `oauth/` - Token issuance, code issuance, JWT handling
- `validators/` - Input validation (authorize, token, email, password, etc.)
- `config/` - Configuration from environment variables
- `constants/` - Audit event names, resource identifiers

### Auth Server (`src/authserver/`)
- `internal/handlers/` - HTTP handlers (auth flows, token, userinfo, DCR)
- `internal/handlers/accounthandlers/` - User self-service handlers
- `internal/handlers/apihandlers/` - Admin API handlers
- `internal/server/routes.go` - All route definitions
- `web/template/` - HTML templates
- `tests/integration/` - Integration tests

### Admin Console (`src/adminconsole/`)
- `internal/handlers/` - Admin UI handlers
- `web/template/` - Admin UI templates

## Database Pattern

Single `Database` interface (`src/core/data/database.go`) with per-DB implementations:
- All methods accept `tx *sql.Tx` (nil = no transaction)
- Uses `sqlbuilder` for query building with DB-specific flavors
- Schema in `src/core/data/sqlitedb/schema.sql`

**Supported**: SQLite, MySQL, PostgreSQL, SQL Server

## OAuth2 Flows Supported

For full documentation, see `site/` (Astro-based docs site).

### Authorization Code (with PKCE)
Primary flow for web/mobile apps. User authenticates via browser, receives code, exchanges for tokens.
- Endpoint: `GET /auth/authorize` → `POST /auth/token` (grant_type=authorization_code)
- PKCE: Configurable globally (`Settings.PKCERequired`) or per-client (`Client.PKCERequired`)
- Supports `response_type=code` with optional `code_challenge` + `code_challenge_method`
- Implementation: `handler_authorize.go`, `handler_token.go`, `oauth/code_issuer.go`

### Client Credentials
Server-to-server auth. No user context, client authenticates directly for access token.
- Endpoint: `POST /auth/token` (grant_type=client_credentials)
- Requires: `Client.ClientCredentialsEnabled = true`
- Auth methods: `client_secret_basic` (Authorization header) or `client_secret_post` (form body)
- Implementation: `handler_token.go` case "client_credentials"

### Refresh Token
Exchange refresh token for new access/refresh tokens. Works with auth code and ROPC flows.
- Endpoint: `POST /auth/token` (grant_type=refresh_token)
- Offline tokens: Configurable idle timeout and max lifetime per client/globally
- Revocation: Old refresh token revoked on use, new one issued
- Implementation: `handler_token.go` case "refresh_token", `oauth/token_issuer.go`

### Implicit Flow (Deprecated)
Legacy flow returning tokens directly in redirect URI fragment. **Deprecated in OAuth 2.1.**
- Endpoint: `GET /auth/authorize` with `response_type=token|id_token|id_token token`
- Disabled by default. Enable via `Settings.ImplicitFlowEnabled` or `Client.ImplicitGrantEnabled`
- Security risk: Tokens exposed in browser history/Referer headers
- Implementation: `handler_auth_issue.go`, `validators/authorize_validator.go`

### ROPC - Resource Owner Password Credentials (Deprecated)
Direct username/password exchange for tokens. **Deprecated in OAuth 2.1** due to credential exposure.
- Endpoint: `POST /auth/token` (grant_type=password, username, password)
- Disabled by default. Enable via `Settings.ResourceOwnerPasswordCredentialsEnabled` or per-client
- Rate limited. Blocks users with 2FA enabled. Logs `AuditROPCAuthFailed` on failure
- Implementation: `handler_token.go` case "password", `validators/token_validator.go`

### Dynamic Client Registration (RFC 7591)
Programmatic client registration for MCP servers, native apps, etc.
- Endpoint: `POST /connect/register`
- Disabled by default. Enable via `Settings.DynamicClientRegistrationEnabled`
- Creates public or confidential clients based on `token_endpoint_auth_method`
- Rate limited. Returns client_id and client_secret (if confidential)
- Implementation: `handler_dynamic_client_registration.go`

## Authentication Flow (Authorization Code)

The auth code flow uses a state machine tracked in `AuthContext` (stored in session cookie).

### ACR Levels (Authentication Context Class Reference)
Defined in `src/core/enums/enums.go`:
- **`urn:goiabada:level1`** - Password only (single factor)
- **`urn:goiabada:level2_optional`** - Password + OTP if user has OTP enabled (skip if not)
- **`urn:goiabada:level2_mandatory`** - Password + OTP required (user must enroll if not already)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leodip/goiabada](https://github.com/leodip/goiabada) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
