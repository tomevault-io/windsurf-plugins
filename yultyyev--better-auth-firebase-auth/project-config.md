---
trigger: always_on
description: This document provides additional context for AI assistants working on this Better Auth Firebase Auth plugin project. **Please read [README.md](./README.md) first** for project overview and features.
---

# AI Assistant Guidelines

This document provides additional context for AI assistants working on this Better Auth Firebase Auth plugin project. **Please read [README.md](./README.md) first** for project overview and features.

**Note:** All planned phases are complete. The plugin is fully implemented and ready for use.

## Quick Reference

- **Main Documentation:** See [README.md](./README.md) for project overview
- **Contributing Guidelines:** See [Better Auth Contributing Guide](https://www.better-auth.com/docs/reference/contributing)
- **Plugin Architecture:** See [Better Auth Plugin Guide](https://www.better-auth.com/docs/guides/your-first-plugin)

## Current Project State

**Phase 1: Project Foundation** ✅ Complete
**Phase 2: Types and Core Structure** ✅ Complete
**Phase 3: Server Plugin - Endpoints** ✅ Complete
**Phase 4: Server Plugin - Hooks** ✅ Complete
**Phase 5: Client Plugin - Methods** ✅ Complete
**Phase 6: Tests** ✅ Complete
**Phase 7: CI/CD and Example Project** ✅ Complete

## Supported Firebase Auth Providers

The plugin currently implements a subset of Firebase Authentication methods:

### Implemented Providers

- **Google OAuth** (`/firebase-auth/sign-in-with-google` endpoint)
- **Email/Password** (`/firebase-auth/sign-in-with-email` endpoint)
- **Password Reset** with email verification and custom URLs
  - `/firebase-auth/send-password-reset` - Send reset email
  - `/firebase-auth/verify-password-reset-code` - Verify reset code
  - `/firebase-auth/confirm-password-reset` - Complete password reset

### Not Implemented (but available in Firebase Auth)

**Social Providers:**
- Facebook, GitHub, Twitter/X, Microsoft, Apple, Yahoo, LinkedIn

**Phone/SMS Authentication:**
- Phone number sign-in with SMS verification
- Multi-factor authentication (MFA)

**Other Methods:**
- Anonymous authentication
- Custom authentication tokens
- SAML/OIDC providers
- Game Center (iOS), Play Games (Android)

### Key Implementation Pattern

All authentication methods follow the same core flow:

1. **Get Firebase ID token** (client-side or server-side)
2. **Verify token** with Firebase Admin SDK (`adminAuth.verifyIdToken()`)
3. **Create/update Better Auth user** via `internalAdapter.createUser()` / `internalAdapter.updateUser()`
4. **Create Better Auth session** via `internalAdapter.createSession()`
5. **Store account link** via `adapter.createAccount()` or `adapter.upsertAccount()` with `provider: "firebase"`

When adding new providers, follow the `signInWithGoogle` endpoint pattern as a reference implementation.

### Important Notes

- All Firebase authentication methods use `provider: "firebase"` in account records
- `providerAccountId` should be the Firebase UID
- User operations must use `internalAdapter` (not `adapter`) for proper database hooks and secondary storage support
- Account operations use `adapter` directly

## Project Files

The project is complete and includes:
- **Configuration files:** `package.json`, `tsconfig.json`, `tsconfig.build.json`, `vitest.config.ts`, `biome.json`
- **Build tooling:** `.gitignore`, `.releaserc.json`
- **Documentation:** `README.md`, `AGENTS.md`, `LICENSE`
- **Source code:**
  - `src/types.ts` - TypeScript interfaces and types
  - `src/firebase-auth-plugin.ts` - Server plugin with endpoints and hooks
  - `src/firebase-auth-client-plugin.ts` - Client plugin with methods
  - `src/index.ts` - Main exports
- **Tests:**
  - `src/firebase-auth-plugin.test.ts` - Server plugin tests (14 tests)
  - `src/firebase-auth-client-plugin.test.ts` - Client plugin tests (10 tests)
- **CI/CD:** GitHub Actions workflows for testing and releases
- **Example:** Minimal Next.js example project in `examples/minimal/`

## Project Structure

```
src/
  firebase-auth-plugin.ts      # Server plugin implementation (endpoints and hooks complete)
  firebase-auth-client-plugin.ts # Client plugin implementation (methods complete)
  index.ts                      # Export both plugins and types
  types.ts                      # Plugin-specific types and interfaces
examples/
  minimal/                      # Minimal Next.js example project
.github/
  workflows/
    release.yml                 # CI/CD release workflow
    ci.yml                      # CI workflow for PRs
```

## Key Implementation Patterns

### Server Plugin Structure

- Export a function that returns a `BetterAuthPlugin`
- Plugin must have unique `id: "firebase-auth"`
- Use `createAuthEndpoint` from `better-auth/api` for endpoints
- Prefer `createAuthMiddleware` from `better-auth/api` for hooks
- Keep fallback compatibility with `better-auth/plugins` for older Better Auth versions
- Follow Better Auth plugin patterns from [plugin guide](https://www.better-auth.com/docs/guides/your-first-plugin)

### Client Plugin Structure

- Export a function that returns a `BetterAuthClientPlugin`
- Use `$InferServerPlugin` to infer types from server plugin
- Use `getActions` to provide client-side methods
- Methods should accept one data argument and optional `fetchOptions`

### Account Storage

- All Firebase Auth methods use `provider: "firebase"` in account records

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yultyyev/better-auth-firebase-auth](https://github.com/yultyyev/better-auth-firebase-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
