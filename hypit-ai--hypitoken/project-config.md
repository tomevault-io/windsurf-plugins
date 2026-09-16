---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

hypitoken (Go module still named `CPA-Claude`, binary `bin/hypitoken`) is a Go reverse-proxy that fans client requests across multiple upstream Anthropic / OpenAI credentials (OAuth + API keys) on **two proxy endpoints** — Claude on `:8317` (`/v1/messages`), Codex on `:8318` (`/v1/chat/completions`, `/v1/responses`) — plus an optional **发卡网 (shop)** listener on `:8319`. On top of the raw proxy sit two optional product layers: a **SaaS multi-tenant billing layer** (`/api/v2/*`, user accounts + USD wallet) and the embedded **admin/landing/console SPA**.

The reusable proxy core (credential pool, usage ledger, pricing, client tokens, request log, rate limiting, the CC mimicry/sidecar fingerprint, and thinking-signature handling) lives in the external module **`github.com/wjsoj/cc-core`**. This repo is the application layer that wires those pieces into endpoints and adds SaaS, shop, and the admin panel. **CPA-Claude (`/home/wjs/Documents/project/Go/CPA-Claude`) is a sibling fork that consumes the same cc-core** — fingerprint/mimicry/sidecar changes land in cc-core only (both forks import `cc-core/{mimicry,sidecar}` directly), then both bump the dependency.

Derivative of [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) (MIT). The Anthropic OAuth refresh, Codex JWT parsing, and uTLS Chrome transport originated upstream and now live in cc-core.

## Build & run

```bash
make build              # build admin SPA (bun) + Go binary into bin/hypitoken
make web-dev            # Vite dev server with API proxy to :8317 (frontend hot reload)
make tidy               # go mod tidy
make lint               # all linters: golangci-lint (Go) + Biome (admin SPA)
make lint-go            # golangci-lint run ./...   (config: .golangci.yml, v2 schema)
make lint-web           # Biome check over internal/admin/web/src
make fmt                # auto-format: golangci-lint fmt (Go) + Biome write (web)
go build ./...          # Go-only build (skips SPA; admin panel falls back to embedded /dist)
go test ./...           # all tests
go test ./internal/server/... -timeout 60s -run TestBootstrap   # sidecar suite (~23s live timing)
```

**Linting is a CI gate (blocking).** `.github/workflows/ci.yml` runs `lint-go` (golangci-lint v2) and `lint-web` (Biome) as separate required jobs alongside `build`. Keep both green:
- **Go** — `golangci-lint` **v2** (`.golangci.yml` is v2-schema; the local binary must be v2.x — v1 cannot parse a go1.25 module). Intentional exceptions live as documented `exclusions.rules` in `.golangci.yml` (Z-Pay MD5, sidecar timing `math/rand`, Stripe `client_secret`, the deliberately-unwired Datadog sidecar) or per-line `//nolint:<linter> // reason`.
- **Web** — **Biome** does both lint + format (replaces ESLint/Prettier), config at `internal/admin/web/biome.json`. Run via `bun run lint` / `bun run lint:fix` / `bun run format`. Strict `recommended` ruleset, zero warnings allowed. `catch` blocks use `errMsg`/`errStatus` from `@/lib/utils` (no `e: any`); shared API shapes live in `@/lib/types`. Hook-dependency exceptions use a single-line `// biome-ignore lint/correctness/useExhaustiveDependencies: reason` directly above the dependency array.

The admin SPA at `internal/admin/web/` (**React 18 + React Router 7 + Vite + Tailwind 4 + shadcn/Radix + R3F**, managed with **bun, not npm**) is built into `internal/admin/web/dist/` and embedded via `//go:embed` in `internal/admin/admin.go`. The `//go:generate` directive there runs `bun install --frozen-lockfile && bun run build`. CI calls `make web` before `go build`, so the SPA is mandatory in releases. The same `dist` is re-served at `/` by the SaaS adapter when SaaS is enabled.

`make build` requires `bun` on PATH. Plain `go build ./...` works without bun if `internal/admin/web/dist/` already contains a build (or the embedded asset can be empty for backend-only iteration).

## The cc-core boundary (read this first)

`internal/` contains almost no credential/usage/pricing logic — those are cc-core packages imported and wired here. The server package imports: `cc-core/{auth, clienttoken, clientguard, pricing, ratelimit, requestlog, usage, thinkingsig, mimicry, sidecar}`. So when a path below says `auth.Pool` / `usage.Store` / `pricing.Catalog` / `clienttoken.Store` / `requestlog.Writer` / `mimicry.SimIdentity` / `sidecar.Manager`, the **type lives in cc-core**, not this repo. There is no `internal/auth/` directory any more.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hypit-ai/hypitoken](https://github.com/hypit-ai/hypitoken) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
