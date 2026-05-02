---
trigger: always_on
description: This reference captures the conventions, architecture, and workflow knowledge that AI agents need when contributing to the Rapidly codebase. Treat it as your onboarding guide to the project.
---

# Rapidly — Agent Development Reference

This reference captures the conventions, architecture, and workflow knowledge that AI agents need when contributing to the Rapidly codebase. Treat it as your onboarding guide to the project.

## Architecture Overview

Rapidly is a file-sharing and paid-content distribution platform. The codebase is structured as a monorepo with three main areas:

### Backend (`server/`)

A Python application powered by **FastAPI**. Core source lives under `server/rapidly/`, organized into domain modules. **PostgreSQL** is the primary data store, accessed through **SQLAlchemy** (ORM). Database models are centralized in `server/rapidly/models` rather than scattered across modules. Background processing uses **Dramatiq** workers.

### Frontend (`clients/`)

A **Next.js** application orchestrated with **Turborepo** and **pnpm**:

- `clients/apps/web/` — the main dashboard where users manage channels, shares, and payments.
- `clients/packages/ui/` — reusable React component library (Radix UI + Tailwind CSS).
- `clients/packages/client/` — auto-generated TypeScript API client and data-fetching hooks.

### Supporting Directories

- `dev/` — development scripts and tooling.
- `docs/` — developer and user documentation, built with Mintlify (https://mintlify.com/docs/llms.txt).

## File Sharing Domain

Rapidly's core domain revolves around secure file sharing with optional monetization. Key concepts:

- **Channels**: A channel is a container owned by a user or organization. It groups related shares together and defines access policies and payment settings.
- **Paid Shares**: Files or content bundles that require payment before the recipient can access them. Payments flow through Stripe Connect, with the channel owner as the connected account.
- **Secret Exchange**: A protocol for sharing sensitive content that ensures data is encrypted in transit and at rest. The server never holds plaintext secrets — it only brokers the handshake.
- **WebRTC Signaling**: For direct peer-to-peer file transfers, the backend acts as a signaling server. It coordinates the connection setup between sender and receiver without relaying the actual file data.

## Authentication

The backend implements a custom auth layer on top of FastAPI's dependency injection.

### Core Concepts

- **`AuthSubject[T]`** — represents the authenticated caller. `T` may be `User`, `Organization`, `Customer`, or `Anonymous`. Inject it into endpoint signatures; endpoints without an `auth_subject` parameter are publicly accessible.
- **Scopes** — fine-grained permissions attached to each `AuthSubject`. An `Authenticator` declares which scopes it requires; access is granted when the subject holds at least one matching scope.

### Per-Module Authenticators

Most modules define their own authenticators in an `auth.py` file, specifying the required scopes and allowed subject types:

```python
# server/rapidly/share/auth.py
_ShareWrite = Authenticator(
    required_scopes={Scope.web_default, Scope.shares_write},
    allowed_subjects={User, Organization},
)
ShareWrite = Annotated[AuthSubject[User | Organization], Depends(_ShareWrite)]
```

### Built-in Authenticator Shortcuts

For endpoints tied to the web dashboard or internal backoffice, use the predefined dependencies from `server/rapidly/auth/dependencies.py`:

- `WebUser` — requires a logged-in user (`AuthSubject[User]`).
- `WebUserOrAnonymous` — accepts either an authenticated user or an anonymous visitor.
- `AdminUser` — restricted to users with admin privileges.

### Usage Example

```python
from rapidly.models import User
from rapidly.share.auth import ShareWrite

@router.post("/shares")
def create_share(auth_subject: ShareWrite) -> Share:
    # Access restricted to users/orgs with web_default or shares_write scope
    ...
```

### Credential Resolution Order

The system resolves the caller's identity by checking, in order: customer session token, member session token, user session cookie, Workspace Access Token, OAuth2 token. When none match, the subject defaults to `Anonymous`. The endpoint's authenticator then decides whether the resolved subject type and scopes are sufficient.

## Code Quality Standards

- Write self-documenting code — add comments only when the intent cannot be expressed through naming and structure.
- Choose descriptive names for variables, functions, and classes.
- Adhere to established conventions and industry best practices.
- All new code should be maintainable and follow SOLID principles.
- Limit changes to what the current task requires; avoid drive-by refactors of unrelated code.

## Backend Development

### Module Layout

Each domain lives in its own directory under `server/rapidly/`. A typical module includes:

| File | Purpose |
|------|---------|
| `endpoints.py` | FastAPI route handlers |
| `service.py` | Business logic, encapsulated in service classes |
| `schemas.py` | Pydantic models for request/response validation |
| `repository.py` | Database access through SQLAlchemy queries |

SQLAlchemy models are the exception — they live centrally in `server/rapidly/models` instead of inside each module.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rapidly-tech/rapidly](https://github.com/rapidly-tech/rapidly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
