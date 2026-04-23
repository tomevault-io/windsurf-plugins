---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BLUE VerifierPlus (fork of Verifier Plus) is a Next.js 15 application for verifying and displaying W3C Verifiable Credentials (VCs). It supports VC Data Model v1/v2, Open Badges 3.0, VPQR, JWT-VC, and SD-JWT formats. It integrates with EBSI/BLUE trust registries and supports OID4VP verifier-initiated credential requests. It also provides online credential storage with public link sharing via MongoDB. Forked from https://github.com/digitalcredentials/verifier-plus, extended for DC4EU. Branded for BLUE, the academic trust network operated by RedIRIS (Red.es) in collaboration with CRUE Universidades Españolas.

## Commands

```bash
npm run dev          # Dev server with Turbopack (http://localhost:3000)
npm run build        # Production build (standalone output)
npm start            # Production server
npm run test         # Playwright tests (DesktopChrome + API projects)
```

Run tests against a specific URL:
```bash
PLAYWRIGHT_TEST_URL=https://stage.verifierplus.org npm run test
```

Run a single test file:
```bash
npx playwright test tests/api.spec.ts --project=API
npx playwright test tests/credential.spec.ts --project=DesktopChrome
```

## Architecture

**Next.js App Router** with `app/` directory. Standalone output mode for Docker deployment.

### Key Directories
- `app/components/` — React UI components (each in its own folder with `.tsx` + `.module.css`)
- `app/lib/` — Shared utilities (verification, database, decoding, registry)
- `app/lib/crypto/` — ES256, JWT-VC, SD-JWT, JWE cryptographic operations (@noble/curves)
- `app/lib/ebsi/` — DID resolver (did:ebsi, did:blue, did:jwk, did:key P-256, did:web), trust registry client, network config
- `app/lib/oid4vp/` — OID4VP verifier logic, session management (LRU cache), protocol types
- `app/api/` — API route handlers
- `app/api/oid4vp/` — OID4VP endpoints (session CRUD, request serving, response handling)
- `app/types/` — TypeScript type definitions (credential.d.ts, presentation.d.ts)
- `tests/` — Playwright E2E tests with visual regression snapshots

### API Routes
- `POST /api/verify` — Verify a credential (JSON-LD, JWT-VC via `{ rawJwt }`, or SD-JWT)
- `POST /api/credentials` — Store a credential (requires signed VP with holder DID)
- `GET /api/credentials/[publicCredentialId]` — Retrieve stored credential
- `DELETE /api/credentials/[publicCredentialId]` — Unshare credential (soft delete)
- `POST /api/proxy` — CORS proxy for fetching external VCs
- `GET /api/exchanges/[txId]` — Verifiable Presentation Request exchange
- `POST /api/oid4vp/session` — Create OID4VP session (returns QR code URI)
- `GET /api/oid4vp/session/[sessionId]` — Poll OID4VP session status
- `GET /api/oid4vp/request/[sessionId]` — Serve authorization request JWT to wallets
- `POST /api/oid4vp/response` — Receive wallet response (direct_post / JARM)
- `GET /api/healthz` — Health check

### Verification Flow
1. User provides input (paste JSON/JWT/SD-JWT, URL via `/#verify?vc=<url>`, QR scan, file upload, or OID4VP)
2. `lib/decode.ts` parses input → detects format (JSON-LD, JWT, SD-JWT) → extracts VC
3. `api/verify` routes to either:
   - `@digitalcredentials/verifier-core` for JSON-LD VCs
   - `lib/validate-jwt.ts` → `verifyJwtVcCredential` or `verifySdJwtCredential` for JWT/SD-JWT
4. JWT/SD-JWT path: DID resolution (`lib/ebsi/didResolver.ts`) → ES256 signature check → EBSI/BLUE TIR trust lookup → expiration check
5. `lib/useVerification.ts` hook + `verificationContext.ts` manage UI state
6. Results displayed via `CredentialVerification`, `VerificationCard`, `ResultLog`, `TrustRegistryCard` components

### OID4VP Flow
1. User selects credential type + response mode → generates QR code via `POST /api/oid4vp/session`
2. Wallet scans QR → fetches authorization request JWT from `GET /api/oid4vp/request/[sessionId]`
3. Wallet presents credential → `POST /api/oid4vp/response` (direct_post or JARM encrypted)
4. Server decrypts (if JARM), extracts VC from VP token, verifies credential
5. Frontend polls `GET /api/oid4vp/session/[sessionId]` → displays result

### State Management
React Context API (`verificationContext.ts`) for verification state. URL hash-based client-side routing (`#verify`, `#/`).

### Database
MongoDB with LRU-cached connection pool (`lib/database.ts`). Credentials stored with UUID, soft-deleted via `shared: false` flag.

## Path Aliases

Use `@/` prefix for imports: `@/components/*`, `@/lib/*`, `@/types/*`, `@/css/*`.

## Styling

CSS Modules (`.module.css`) per component + Bootstrap + Tailwind CSS 4. Dark mode via `.darkmode` class. CSS custom properties for theming (`--primary`, `--text`, etc.) defined in `globals.css`.

## Environment Variables

See `.env.example`. MongoDB connection (`DB_USER`, `DB_PASS`, `DB_HOST`, `DB_NAME`, `DB_COLLECTION`), deployment URL, and exchange server URL. Required for credential storage features.

OID4VP/EBSI/BLUE optional overrides:
- `OID4VP_VERIFIER_DID` — Verifier's DID (defaults to deployment URL)
- `OID4VP_SESSION_TTL` — Session timeout in ms (default: 300000 = 5 min)
- `EBSI_DID_REGISTRY_URL`, `EBSI_TIR_URL` — Override EBSI endpoints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LAArino) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
