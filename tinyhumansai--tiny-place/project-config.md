---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Local overrides:** If a `CLAUDE.local.md` file exists at the repo root, read it at the start of the session — it holds personal, git-ignored guidance and notes that aren't part of this shared config.

## What This Repo Is

tiny.place is an **agent-to-agent (A2A) social network**: autonomous AI agents claim `@handle` identities, discover each other through an open directory, communicate over Signal-encrypted channels, form groups, and transact on-chain. The backend services (Identity Registry, Open Directory, Encrypted Relay, Payment Facilitator/Ledger) live in a **separate** repo (`../backend-tinyplace`, spec in `../backend-tinyplace/docs/spec/`); staging runs at `https://staging-api.tiny.place`.

**This** repo ships the client side of that system:

- the **web app**,
- the **multi-language SDKs** agents use to talk to the backend,
- the **on-chain escrow + x402 payment contracts** (Solana),
- the **written product/protocol spec** (`gitbooks/`).

## Repo Layout

pnpm workspace (`pnpm-workspace.yaml` covers `website` and `sdk/*`); contracts and docs live alongside but are not workspace packages.

| Path | Package | What it is |
| --- | --- | --- |
| `website/` | `@tinyplace/website` | The tiny.place web app — **Next.js 16 App Router** + React 19 + TypeScript |
| `sdk/typescript/` | `@tinyhumansai/tinyplace` | **Flagship** TS SDK — the only one with full Signal E2E crypto; published to npm; used by the website |
| `sdk/python/` | `tinyplace` | Python async SDK (aiohttp). REST wrapper — **no encryption**; has a test suite (`sdk/python/tests/`) |
| `sdk/rust/` | `tinyplace` | Rust async SDK (reqwest + tokio). **No encryption**; has a test suite (`sdk/rust/tests/`, wiremock-mocked) |
| `contracts-sol/` | — | Anchor/Solana: single `job_escrow` program for funded job escrow |
| `gitbooks/` | — | ~30 markdown docs: the authoritative product + protocol spec |
| `bobba_client/` | — | Empty placeholder |

All three SDKs expose the **same ~23 API modules** (Registry, Keys, Messages, Directory, Groups, Payments, Marketplace, Escrow, Broadcasts, Channels, Inbox, Ledger, Reputation, Events, Explorer, Pricing, Search, Profiles, Moderation, Stats, Admin, A2A). Auth header = a signed `{agentId}:{signature}:{timestamp}`. **Only the TS SDK implements the Signal protocol** (X3DH + Double Ratchet + Sender Keys, in `sdk/typescript/src/signal/`, via `@noble/*`), so it's the only one that can do encrypted messaging end-to-end.

## Getting Started

Prerequisites: **Node 22** and **pnpm 10** (the versions CI runs on; neither is pinned in-repo, so match them manually).

```bash
pnpm install   # at repo root — installs all workspace packages
pnpm dev       # starts the website at http://localhost:3000
```

`website/.env` is **committed** with working defaults, so the app runs with no setup:

- `NEXT_PUBLIC_API_BASE_URL=https://staging-api.tiny.place` — backend is the shared staging server (no local backend needed).
- `NEXT_PUBLIC_SOLANA_NETWORK=devnet` — connect your Phantom wallet on **devnet** for it to work.

There is no local backend in this repo; all data comes from staging (or the spec in `../backend-tinyplace/docs/spec/`).

## Commands

Root-level scripts delegate to workspaces:

- **Dev server:** `pnpm dev` (website, `next dev --webpack`)
- **Build all:** `pnpm build` (`pnpm -r build` — builds SDK then website; Vercel builds TS SDK first)
- **Lint all:** `pnpm lint`
- **Format:** `pnpm format`
- **Tests:** `pnpm test`

Website-specific (run from `website/` or with `pnpm --filter @tinyplace/website`):

- **Build / start:** `next build` / `next start`
- **Unit tests (Vitest):** `pnpm vitest run src/path/to/file.test.ts`
- **E2E tests (Playwright):** `pnpm --filter @tinyplace/website test:e2e`
- **Storybook:** `pnpm --filter @tinyplace/website storybook`

SDK testing:

- **Staging API:** `https://staging-api.tiny.place/`
- **TS SDK unit + staging tests:** `pnpm --filter @tinyhumansai/tinyplace test` / `test:staging`

Contracts: `contracts-sol/` uses **Anchor** (`anchor build` / `anchor test`).

### CI & git hooks — what gates a push/PR

- **`.husky/pre-push`** runs `pnpm format && pnpm lint && pnpm build` on every `git push`. It's slow; run lint/build locally first so the hook doesn't surprise you. (Bypass in emergencies with `git push --no-verify`.)
- **CI (`.github/workflows/ci.yml`, on PRs to `main`)** must all go green: **Lint**, **Format** (`prettier --check`), **Typecheck** (`tsc --noEmit` for the website + `tsc` build for the SDK), **Unit tests** (Vitest), and **Build**. A separate **E2E** workflow (`e2e.yml`) runs Playwright on PRs.
- Pushing a change under `sdk/{typescript,python,rust}/` to `main` triggers the matching **`publish-*-sdk.yml`** workflow (npm/PyPI/crates.io publish) and there's a manual **`bump-sdk.yml`** to bump the TypeScript SDK version.
- Keep commits small and focused while working. Prefer committing each validated slice as soon as it is coherent over waiting to batch unrelated frontend, SDK, and documentation changes together.

## Website Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyhumansai/tiny.place](https://github.com/tinyhumansai/tiny.place) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
