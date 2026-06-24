---
trigger: always_on
description: Shellgate is a SvelteKit monolith — dashboard + API + gateway in one process.
---

# Development Guidelines

## Architecture

Shellgate is a SvelteKit monolith — dashboard + API + gateway in one process.

### Layers

```
src/lib/server/services/   ← All business logic lives here
src/lib/server/utils/       ← Pure helpers (CIDR, URL validation, etc.)
src/routes/api/             ← Admin API (thin wrappers calling services)
src/routes/(app)/           ← Dashboard pages (call services directly, no HTTP to self)
src/routes/gateway/         ← Agent-facing: HTTP proxy to upstream APIs
src/routes/ssh/             ← Agent-facing: SSH command execution
src/routes/discovery/       ← Agent-facing: list accessible targets
src/routes/bootstrap/       ← Agent-facing: full session-start context (REST)
```

### Key services

| Service | Purpose |
|---|---|
| `targets` | CRUD for API, SSH, and email targets |
| `tokens` | API key generation, hashing, revocation |
| `permissions` | Token ↔ target access control |
| `auth-methods` | Credentials stored per target (bearer, basic, custom_header, ssh_key, imap_smtp) |
| `gateway` | Proxy logic: resolve target, inject auth, forward request |
| `ssh` | SSH command execution via `ssh2` |
| `mail` | Email operations via ImapFlow (IMAP) + Nodemailer (SMTP) |
| `audit` | Request logging to `audit_logs` table |
| `users` | Dashboard user management (email + password) |
| `webhook-endpoints` | CRUD for incoming webhook endpoint registrations |
| `webhook-events` | Event creation, polling, ACK, cleanup |
| `connected-accounts` | OAuth account linking, managed target provisioning, token refresh |

### Data model

```
tokens ──┐
         ├── token_permissions (unique: token + target)
targets ──┘
  │
  └── target_auth_methods (one default per target)

tokens ──── webhook_endpoints (one token can have multiple endpoints)
              │
              └── webhook_events (pending/delivered/expired)

connected_accounts ──── targets (via connected_account_id FK, cascade delete)

users (dashboard login)
audit_logs (every gateway + SSH + mail request)
```

- Targets have `type: "api" | "ssh" | "email"`. API targets have `baseUrl`, SSH targets have `config` (JSONB: host, port, username), email targets have `config` (JSONB: imap + smtp server settings) and `email` (mailbox address).
- Targets may be **managed** by a connected account (`connected_account_id` set). Managed targets are read-only — only permissions can be changed. They have a `capability` field (`mail` or `calendar`).
- Cascade deletes: deleting a target removes its auth methods and permissions. Deleting a connected account removes its managed targets.
- Auth method types: `bearer`, `basic`, `custom_header`, `query_param`, `ssh_key`, `jwt_es256`, `oauth2_refresh_token`, `json_body`, `imap_smtp`.
- Webhook endpoints are linked to tokens (agents), not targets. Each endpoint has a unique slug for its public URL.
- Webhook events expire after 7 days. Agents poll and ACK events.
- Webhook endpoints have optional `handlingInstructions` (plain text) that agents receive in the poll response.

### Agent-facing routes

These are NOT behind dashboard auth — they use bearer token auth (`requireBearer`):

| Route | Purpose |
|---|---|
| `GET /bootstrap` | Full session-start context: targets, skills, webhooks, memories, wiki pages |
| `GET /discovery` | List targets accessible to this token |
| `ALL /gateway/[target]/[...path]` | Proxy HTTP to upstream API, inject stored credentials |
| `POST /ssh/[target]/exec` | Execute command on SSH target, return stdout/stderr/exitCode |
| `POST /mail/[target]/search` | Search emails in a mailbox |
| `GET /mail/[target]/message/[id]` | Read full email by UID |
| `GET /mail/[target]/message/[id]/attachment/[partId]` | Download email attachment |
| `POST /mail/[target]/send` | Send email (requires approval) |
| `POST /mail/[target]/draft` | Create draft email |
| `GET /mail/[target]/folders` | List mailbox folders |
| `POST /mail/[target]/move` | Move email to folder |
| `POST /mail/[target]/flag` | Set/unset email flags |
| `GET /health` | Health check |
| `GET /verify-connection` | Connection verification for agent setup |
| `GET /api/skill` | Returns OpenClaw skill YAML |
| `POST /api/install/openclaw` | OpenClaw integration installer |
| `POST /api/install/claude-code` | Claude Code integration installer |
| `POST /api/install/codex` | Codex CLI integration installer |
| `POST /webhooks/incoming/[slug]` | Receive webhook from external service |
| `GET /webhooks/poll` | Poll pending webhook events for this token |
| `POST /webhooks/ack` | Acknowledge processed webhook events |
| `POST /webhooks/endpoints/[id]/instructions` | Save handling instructions for endpoint |
| `POST /mcp` | MCP server (Streamable HTTP transport) |

### Dashboard routes

Behind session auth (cookie-based):

| Route | Purpose |
|---|---|
| `/` | Dashboard overview (stats) |
| `/targets` | Manage API + SSH + email targets |
| `/targets/[slug]` | Target detail + auth methods |
| `/api-keys` | Manage agent tokens + permissions |
| `/api-keys/[id]` | Token detail |
| `/logs` | Audit log viewer |
| `/connect` | Agent connection flow |
| `/webhooks` | Manage incoming webhook endpoints |
| `/webhooks/[id]` | Webhook detail + events + handling instructions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthiastjong/shellgate-legacy](https://github.com/matthiastjong/shellgate-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
