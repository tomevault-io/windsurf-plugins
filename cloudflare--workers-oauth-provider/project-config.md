---
trigger: always_on
description: `@cloudflare/workers-oauth-provider` is a production-grade OAuth 2.1 provider library for Cloudflare Workers. It implements authorization code flow with PKCE, dynamic client registration, token exchange, and end-to-end encryption of sensitive data stored in KV.
---

# AGENTS.md

## Project overview

`@cloudflare/workers-oauth-provider` is a production-grade OAuth 2.1 provider library for Cloudflare Workers. It implements authorization code flow with PKCE, dynamic client registration, token exchange, and end-to-end encryption of sensitive data stored in KV.

**Primary use case:** This library powers authentication for **MCP (Model Context Protocol) servers**. MCP servers are OAuth Resource Servers, and this library provides the authorization server functionality needed to secure them.

This library was largely written with Claude AI assistance, with all code thoroughly reviewed by Cloudflare security engineers.

## MCP specification compliance

When modifying OAuth functionality, **always check the latest published MCP specification** (not drafts):

- **Specification:** https://modelcontextprotocol.io/specification/2025-11-25
- **Authorization section:** https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization

This library must be feature-complete with the latest published MCP spec version. Key MCP auth requirements:

- MCP servers are OAuth Resource Servers with protected resource metadata (RFC 9728)
- MCP clients must support Resource Indicators (RFC 8707) for audience-scoped tokens
- Client registration supports: out-of-band, CIMD (Client ID Metadata Documents), and DCR (Dynamic Client Registration)
- Streamable HTTP transport uses OAuth 2.1 for authentication

When in doubt about OAuth behavior, the MCP specification takes precedence for MCP-related use cases.

## Repository structure

```
workers-oauth-provider/
├── src/
│   └── oauth-provider.ts      # Single source file (~4,600 lines)
├── __tests__/
│   ├── oauth-provider.test.ts # Comprehensive test suite (~9,400 lines)
│   ├── setup.ts               # Vitest setup and mocking
│   └── mocks/
│       └── cloudflare-workers.ts
├── dist/                      # Build output (tsdown)
├── examples/
│   └── typed-env-worker/      # Example worker with typed environment
├── .github/workflows/
│   ├── ci.yml                 # PR validation
│   ├── release.yml            # Changesets-based npm publishing
│   └── pkg-pr-new.yml         # PR preview packages
├── storage-schema.md          # KV namespace data structure docs
├── SECURITY.md                # Vulnerability reporting
└── README.md                  # Usage documentation
```

**Single-file architecture:** All implementation is in `src/oauth-provider.ts`. This is intentional for security review — all code in one place aids auditing.

## Setup

```bash
npm install    # Install dependencies
npm run build  # Build with tsdown
```

Node 24+ required.

## Commands

| Command              | What it does                              |
| -------------------- | ----------------------------------------- |
| `npm run build`      | Builds single-file ESM bundle with tsdown |
| `npm run check`      | Runs typecheck + tests                    |
| `npm run typecheck`  | TypeScript type checking (no emit)        |
| `npm run test`       | Runs vitest test suite                    |
| `npm run test:watch` | Runs vitest in watch mode                 |
| `npm run prettier`   | Formats all files with Prettier           |

## Code standards

### TypeScript

- Strict mode enabled
- Target: ES2021, Module: ES2022
- All public methods and interfaces must have JSDoc documentation
- Private fields use `#` prefix (modern TS private fields)

### Naming conventions

- `PascalCase` for classes, interfaces, enums
- `camelCase` for methods, variables
- `SCREAMING_SNAKE_CASE` for constants
- `Impl` suffix for internal implementations
- `Options` suffix for configuration interfaces

### Architecture patterns

**PImpl pattern:** The public `OAuthProvider` class wraps a private `OAuthProviderImpl`. This prevents TypeScript private methods from being accidentally exposed over RPC in Cloudflare Workers.

```typescript
export class OAuthProvider {
  #impl: OAuthProviderImpl;
  fetch(...) { return this.#impl.fetch(...); }
}
```

**Dual handler support:** The library supports both `ExportedHandler` (plain objects) and `WorkerEntrypoint` (classes) patterns. Maintain both for backwards compatibility.

### Formatting

Prettier with 120 character line width. Run `npm run prettier` before committing.

## Security considerations

This is a security-critical OAuth library. All changes must consider:

**Token storage:**

- Secrets (tokens, authorization codes) are stored as SHA-256 hashes only
- Props are encrypted with AES-GCM, key wrapped with the token itself
- Only token holders can decrypt their associated props

**Validation:**

- Redirect URIs validated against XSS payloads
- Client IDs validated (including CIMD URL validation)
- PKCE enforced for public clients (S256 method)
- Scope downscoping validated per RFC 6749 Section 3.3

**Refresh token rotation:**

- Dual refresh tokens: current + previous both valid
- Handles network failure cases gracefully
- Previous token invalidated only after new token first used

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/workers-oauth-provider](https://github.com/cloudflare/workers-oauth-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
