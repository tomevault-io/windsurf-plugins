---
trigger: always_on
description: Node.js/TypeScript authentication service. Social login (GitHub, Google) via OAuth2 PKCE, RS256 JWT issuance, token refresh/revocation. Relay verifies tokens using the public key endpoint.
---

# Sesori Auth Server

Node.js/TypeScript authentication service. Social login (GitHub, Google) via OAuth2 PKCE, RS256 JWT issuance, token refresh/revocation. Relay verifies tokens using the public key endpoint.

## STRUCTURE

```
src/
├── types/             # Enums + shared types (mongo.ts, oauth.ts)
├── clients/
│   ├── auth/          # OAuth provider abstraction
│   │   ├── oauth-client.ts   # Abstract base — template method: exchangeCode → resolveIdentity
│   │   ├── github-client.ts  # GithubClient extends OAuthClient
│   │   └── google-client.ts  # GoogleClient extends OAuthClient (JWKS verification)
│   └── openai-client.ts      # OpenAI transcription client
├── db/
│   ├── mongo-db-connector.ts  # MongoDbConnector — connection lifecycle, health check
│   └── mongo-db-accessor.ts   # MongoDbAccessor — generic DB access + config-driven ensureIndexes
├── lib/               # Utilities (state-store.ts — LRU singleton, errors.ts — ApiError hierarchy)
├── middleware/         # createAuthMiddleware factory → requireAuth preHandler hook
├── models/            # Zod schemas — api.ts, bridge.ts (shared bridge enums/schemas), documents.ts, jwt.ts
├── repositories/      # Data access — user-repo.ts, oauth-account-repo.ts, bridge-repo.ts, glossary-entry-repo.ts
├── routes/
│   └── auth/          # OAuth + pending-confirmation flow
│       ├── github.ts             # GET /auth/github, POST /auth/github/init, POST/GET callbacks
│       ├── google.ts             # mirror of github.ts for Google
│       ├── init.ts               # Shared helpers: parseSessionTokenHeader, createPendingOAuthInit, …
│       ├── provider-callback.ts  # GET interstitial + POST confirm/deny (HTML responses)
│       └── session-status.ts     # GET /auth/session/status long-poll
├── services/          # Business logic — auth-service.ts, token-service.ts, voice-service.ts
│   └── pending-auth-store.ts     # In-memory LRU of pending OAuth sessions (anti-phishing flow)
├── config.ts          # Zod-validated env config
├── index.ts           # Composition root (wires all dependencies)
└── server.ts          # Fastify app factory (buildApp receives typed AppServices)
```

## WHERE TO LOOK

| Task                       | Location                                                                 | Notes                                                                                       |
| -------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| Add OAuth provider         | `src/clients/auth/` + `src/routes/auth/`                                 | Extend OAuthClient, implement exchangeCode + resolveIdentity, add route plugin in server.ts |
| OAuth pending/confirm flow | `src/routes/auth/init.ts` + `provider-callback.ts` + `session-status.ts` | Anti-phishing interstitial; pending state in `src/services/pending-auth-store.ts`           |
| Modify JWT claims          | `src/models/jwt.ts` + `src/services/token-service.ts`                    | Zod schema defines payload shape                                                            |
| Add API endpoint           | `src/routes/`                                                            | Register as Fastify plugin in `server.ts`, add to AppServices if deps needed                |
| Change DB schema           | `src/models/documents.ts` + `src/repositories/`                          | Zod document schemas, raw MongoDB driver                                                    |
| Add DB collection          | `src/types/mongo.ts` + `src/db/mongo-db-accessor.ts`                     | Add to AuthDbCollection enum + DATABASE_CONFIG                                              |
| Auth middleware            | `src/middleware/auth.ts`                                                 | `createAuthMiddleware(tokenService)` factory                                                |
| Manage bridges             | `src/routes/bridges.ts` + `src/services/bridge-service.ts` + `src/repositories/bridge-repo.ts` + `src/services/bridge-state-tracker.ts` | Per-bridge registry behind `/auth/me bridges[]`; see BRIDGE SUBSYSTEM below                 |
| Wire dependencies          | `src/index.ts`                                                           | Composition root — all instantiation happens here                                           |

## CONVENTIONS

- **DI**: Constructor injection for stateful classes. Composition root in `index.ts`.
- **Validation**: All request/response types defined with Zod — safeParse, no .parse()
- **No ODM**: Raw MongoDB driver. Collections via `MongoDbAccessor.getCollection()`
- **Error handling**: Fastify error handler, ApiError hierarchy in `src/lib/errors.ts`
- **ESM**: `"type": "module"` in package.json
- **Config**: All env vars validated by Zod schema at startup (`src/config.ts`)
- **Secrets**: SOPS + age encryption for env files (`env/app/*.env`). NEVER commit plaintext `.env` or `*.pem`
- **Types**: Shared types in `src/types/`. DB-specific config types stay in `src/db/`.

## SCALING CONSTRAINTS


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sesori-ai/sesori_auth_server](https://github.com/sesori-ai/sesori_auth_server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
