---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## ⚠ Mandatory: Red-Green-Refactor

**Every code change MUST follow the RGR cycle. No exceptions.**

1. Write a failing test BEFORE writing any production code.
2. Write the minimum code to make the test pass.
3. Refactor the code you wrote and touched — this step is NOT optional.

For tasks with no testable behavior (config, docs), the refactor step still applies to any code touched. See [Development Workflow](#development-workflow) for the full sequence including VERIFY, DOCUMENT, and SMOKE.

## Build & Test Commands

```bash
npm test              # Run all unit/integration tests (Vitest)
npm run test:watch    # Run tests in watch mode
npm run test:coverage # Run with V8 coverage (thresholds enforced)
npm run test:e2e      # Run Playwright E2E tests
npm run dev           # Dev server on http://localhost:3000
npm run serve         # Standalone Hono server (self-hosting; build first)
npx tsc --noEmit      # TypeScript type check (strict mode)
```

Run a single test file: `npx vitest run <path/to/file>`.

## Architecture

FeedZero is a privacy-first RSS reader. React + TypeScript UI, Zustand state, React Router, Tailwind CSS v4. Core modules (`src/core/`, `src/utils/`) are framework-agnostic TypeScript with zero React/UI imports — they are the shared backend.

### Runtime Dependencies

- **UI**: React + React DOM (functional components only), React Router, Radix UI + shadcn/ui wrappers in `src/components/ui/` (Button, Dialog, AlertDialog, DropdownMenu, Sheet, Sidebar, etc. — use these, do not build from scratch), lucide-react icons, react-resizable-panels, sonner toasts (`<Toaster>` in `src/app.tsx`, trigger via `toast()`), next-themes, class-variance-authority, clsx + tailwind-merge via `cn()`.
- **State / storage**: Zustand (stores call core modules directly), Dexie.js (IndexedDB, encrypted).
- **Parsing / extraction**: feedsmith (RSS/Atom/JSON Feed + OPML), Defuddle (full-text extraction; pluggable), marked (markdown → HTML; always piped through DOMPurify), DOMPurify (XSS — do not hand-roll).
- **Server**: Hono (14kB, Web standard `Request/Response`; powers self-hosting via `server.ts`).

### Data Flow

Add feed: `feed-service.ts` (normalize, dedup) → `/api/feed` proxy → `validator.ts` → `parser.ts` → `sanitizer.ts` (DOMPurify) → `schema.ts` → `crypto.ts` (AES-GCM-256) → `db.ts` (Dexie) → Zustand → React → URL auto-selects new feed.

Full-text extraction (user-initiated): click "Extracted" → `/api/page` → `extractor.ts` → `defuddle-extractor.ts` → `cleanup.ts` → DOMPurify → cached in extraction store.

### Core Modules (Framework-Agnostic)

- **src/utils/result.ts** — `Result<T>` (`ok`/`err`) used everywhere instead of throwing. `andThen` chains; `fromPromise` wraps async.
- **src/utils/constants.ts** — DB name, crypto params, `LOCAL_STORAGE` keys, default passphrase.
- **src/core/storage/crypto.ts** — PBKDF2 + AES-GCM + HMAC-SHA256 via Web Crypto API.
- **src/core/crypto/argon2.ts** — Argon2id wrapper around `hash-wasm`. Memory-hard KDF used to encrypt new sync vaults so a 4-word diceware passphrase (~51.7 bits) is no longer brute-forceable by GPU farms. Production params: 64 MiB / t=3 / p=1 (OWASP). The vault envelope stamps which KDF derived its key (`KdfSpec` in `src/core/sync/types.ts`) so recovery on a new device picks the matching derivation. Legacy PBKDF2 vaults auto-upgrade to Argon2id the first time the user types their passphrase on any device (recovery-step / switchToExistingCloud → `upgradeVaultKdf`).
- **src/core/storage/db.ts** — Dexie storage. Content AES-GCM encrypted; index fields (url, feedId, guid) HMAC-SHA256 hashed so we can query without exposing plaintext. Call `open(passphrase)` or `openWithKeys(dbKeyJwk, hmacKeyJwk)` first.
- **src/core/storage/key-material.ts** — `deriveAndStoreKeys`, `loadStoredKeys`, `clearStoredKeys`. Derives DB/HMAC/optional vault keys, persists JWK to localStorage. Raw passphrase is never persisted.
- **src/core/storage/schema.ts** — `createFeed()` / `createArticle()` factories returning `Result`.
- **src/core/discovery/** — `discoverFeed(url)` multi-strategy cascade; `strategies.ts` holds the pure functions.
- **src/core/crypto/passphrase-generator.ts** — EFF large wordlist, 4 words, ~51.7 bits entropy.
- **src/core/proxy/validate-url.ts** — SSRF-safe URL validation. Returns `Result<URL>`.
- **src/core/proxy/proxy-handler.ts** — Shared proxy logic for serverless functions.
- **src/core/extractor/extractor.ts** — Public `extract(html, url)` + `needsExtraction(article)`. Swap implementation by changing the import.
- **src/core/extractor/{defuddle-extractor,cleanup,markdown}.ts** — Defuddle impl; HTML cleanup; markdown→HTML via marked + DOMPurify.
- **src/core/extractor/adapters/** — Site-specific adapters. `SiteAdapter` interface, `AdapterRegistry` (O(1) domain lookup). `github-adapter` extracts README; `default-adapter` uses Defuddle.
- **src/core/sync/types.ts** — `VaultData`, `EncryptedVault`, `SyncStorageAdapter`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forcingfx/feedzero](https://github.com/forcingfx/feedzero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
