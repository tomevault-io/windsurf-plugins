---
trigger: always_on
description: - Write clean, readable, and maintainable code
---

# Copilot Instructions

## Code Quality
- Write clean, readable, and maintainable code
- Use design patterns and best practices appropriate for the problem at hand
- Use descriptive, intention-revealing variable and function names
- Apply OOP principles where appropriate; avoid over-engineering
- Follow TDD principles — write unit tests for all new features and bug fixes
- Consider edge cases and potential pitfalls before implementing

## Tech Stack Conventions
- Backend: C# .NET with Clean Architecture (Domain → Application → Infrastructure → API)
- Frontend: React + TypeScript + Tailwind CSS
- Follow CQRS in the Application layer — Commands and Queries handled via MediatR
- Use the repository pattern in Infrastructure; never access `DbContext` directly from controllers
- DTOs belong in the Application layer; never expose domain entities directly from the API
- Use FluentValidation for all request validation

## API & Error Handling
- All API responses follow a consistent envelope: `{ data, error, statusCode }`
- Use ProblemDetails (RFC 7807) for error responses
- Handle all exceptions in global middleware — not in controllers
- Validate inputs at the API boundary; trust data inside the domain

## Security

### Authentication & Authorisation
- JWTs must be short-lived (≤15 min access token); use refresh tokens stored in `HttpOnly`, `Secure`, `SameSite=Strict` cookies — never in `localStorage`
- Enforce `[Authorize]` on every controller/endpoint by default; opt out explicitly with `[AllowAnonymous]` only where required
- Validate the JWT `iss`, `aud`, and expiry on every request — do not accept tokens signed with `none` or HS256 with an empty key
- Implement role-based and resource-based authorisation separately: role gates what you can do, resource ownership gates what you can do it to
- Never return meaningful information in 401 vs 403 responses that could enumerate users or roles

### Input Handling & Injection Prevention
- Never log sensitive data (passwords, tokens, PII, full request bodies containing credentials)
- Always validate and sanitize every user-supplied value at the API boundary before it enters the domain
- Use parameterized queries (EF Core) — never concatenate SQL strings, LINQ expressions derived from raw input, or dynamic `ORDER BY` clauses without an allow-list
- Apply `[MaxLength]`, `[RegularExpression]`, and FluentValidation rules; reject oversized payloads at the middleware level (configure `MaxRequestBodySize`)
- Sanitize filenames and reject path traversal patterns (`../`, `..\`) before using them in file operations or blob storage keys
- Treat every value from `HttpContext.Request` (headers, query strings, route params, body) as untrusted

### Secrets & Configuration
- No secrets in source code or appsettings files committed to git — use Azure Key Vault references (`@Microsoft.KeyVault(...)`) in App Service config
- Rotate secrets on suspected compromise; prefer managed identities over connection strings wherever Azure services support it
- Never echo secrets back in API responses, logs, or error messages
- Use the `.env` / `public/config.js` pattern for frontend runtime config; never bake secrets into the Vite build bundle (`VITE_` prefixed vars are public)

### Transport & Headers
- Enforce HTTPS everywhere; reject HTTP with a 301 permanent redirect
- Set security headers on every response via `SecurityHeadersMiddleware`: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy`
- Scope CORS to the specific SWA origin(s) — never use `AllowAnyOrigin` with `AllowCredentials`
- Enable HSTS with `includeSubDomains` and a minimum `max-age` of 1 year in production

### Rate Limiting & Abuse Prevention
- Apply rate limiting to all authentication endpoints (`/api/auth/login`, `/api/auth/register`, `/api/auth/forgot-password`) — lower limits than general API
- Return `429 Too Many Requests` with a `Retry-After` header; do not leak quota counts in error bodies
- Validate and cap pagination parameters (`pageSize` ≤ 100) to prevent large data dumps

### OWASP Top 10 Checklist
- **A01 Broken Access Control** — always verify the requesting user owns the resource before returning or mutating it
- **A02 Cryptographic Failures** — use BCrypt/Argon2 for passwords; never MD5/SHA-1; encrypt PII at rest
- **A03 Injection** — parameterized queries, HTML-encode all user content rendered in the browser
- **A05 Security Misconfiguration** — disable Swagger UI in production; remove debug endpoints; apply least-privilege IAM roles
- **A07 Identification & Authentication Failures** — enforce MFA for admin accounts; lock accounts after N failed attempts
- **A09 Security Logging & Monitoring** — log all auth events (login, logout, failed attempts, token refresh) with user ID and IP; alert on anomalies

## Frontend

### Architecture & Component Design
- Colocate component tests with the component file (`Component.test.tsx`)
- Encapsulate business logic in custom hooks; keep components purely presentational — no API calls, no business logic directly in JSX
- Prefer `const` arrow functions for all components and hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddhruvgupta/Lawgate-ProjectsOnPhone](https://github.com/ddhruvgupta/Lawgate-ProjectsOnPhone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
