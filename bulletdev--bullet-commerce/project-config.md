---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Overview

bullet-commerce is a production-oriented **e-commerce backend in Go** - the BFF for
the clubedojava stack. It exposes a RESTful API for catalog, cart, orders, stock,
PIX/card payments, shipping and users, over PostgreSQL. Domain-Driven Design over
a Clean Architecture spine: the domain (aggregates, invariants, ports) knows
nothing about HTTP, the pgx driver, or any specific payment/shipping provider.

> The Go module is `bullet-commerce` - imports start with
> `bullet-commerce/internal/...` even though the repo/binary is bullet-commerce.

## Development commands

```bash
go run cmd/main.go              # start server on :4444 (or $PORT)
go build ./...                  # build all packages
go build -mod=vendor -o main cmd/main.go   # production build (vendored)
go test ./...                   # all tests
go test -race ./...             # run before closing any work item
go vet ./...
gofmt -l internal/ cmd/         # must print nothing
golangci-lint run               # govet staticcheck errcheck revive gocritic gosec ineffassign unused gofmt
govulncheck ./...               # CVE scan of Go deps
migrate -database "$DATABASE_URL" -path internal/database/migrations up
docker compose up -d            # app (4444) + postgres
```

## Technology stack

```
Go 1.23 (toolchain go1.24.1) · module bullet-commerce
gorilla/mux v1.8.1        router
pgx/v5 v5.7.4            PostgreSQL 17 driver + pgxpool
golang-jwt/jwt v5.2.2    JWT HS256
golang.org/x/crypto     Argon2id password hashing
google/uuid             IDs
joho/godotenv           .env config
log/slog                structured JSON logging
testify v1.10 + pgxmock/v4   tests (pgxmock for repos, testify/mock for handlers)
CI: golangci-lint · govulncheck · Semgrep · CodeQL · Qodana · dependency-review (Actions SHA-pinned)
```

## Package structure

```
cmd/main.go                  # composition root: config → db → repos → providers → handlers → setupRoutes
internal/
  models/                    # entities + value objects (Order, Product, ProductVariant, Money = int64 cents)
  auth/                      # jwt.go · password.go (Argon2id) · middleware.go (Authenticate + RequireAdmin)
  config/                    # typed 12-factor ENV loader (getEnv/getInt/getBool/getDuration)
  database/                  # pgxpool + migrations/
  middleware/                # RequestID · CORS · BodyLimit(1MiB)
  handlers/                  # HTTP adapters, one per domain - NO business rules
  webutils/                  # WriteJSON · ErrorJSON · ReadJSON
  products/ variants/ cart/ orders/ categories/ users/ addresses/   # aggregate repos: repository.go + repository_mock.go + repository_test.go
  payment/                   # payment.Provider port + Registry
    propay/                  # ProPay/OpenPix adapter (service JWT + HMAC webhook)
  shipping/                  # shipping.Provider port + TableProvider (Brazil region rules)
```

Adding a domain: `internal/{domain}/repository.go` (interface + postgres impl +
`repository_mock.go`) + `repository_test.go`; register handler + route in
`setupRoutes` (`cmd/main.go`).

## Domains

- **products** - Product aggregate root (catalog, search, featured, soft delete).
- **variants** - ProductVariant entity; **owns the stock invariant**
  (Reserve/Claim/Release, atomic per variant). `available = stock - stock_reserved`.
- **categories** - product categorization.
- **cart** - Cart aggregate; **line identity = variant** (same product in two
  sizes = two lines). Optional coupon.
- **orders** - Order aggregate; two state machines (`status` + `payment_status`);
  coordinates variant stock reservation inside its own `pgx.Tx`. Saga planned
  (WI-P5). Background cleanup tickers (pending_payment 30min · unpaid 15min).
- **users** - RBAC role `user | admin`; CPF on profile, required at checkout.
- **addresses** - user addresses; default shipping address.

## Conventions

- **Money = `int64` minor units (cents) + currency, everywhere. Never float.**
  `models/money.go`, `DefaultCurrency = "BRL"`. Formatting to decimal is a
  frontend concern.
- **DDD.** One aggregate → one repository. Cross-aggregate work (order reserving
  variant stock) goes through the variant repository **inside the order's tx** -
  never by reaching into another aggregate's tables. Invariants live with the
  aggregate that owns them (stock is enforced atomically in SQL, not in a handler).
- **Reserve / Claim / Release.** Reserve (hold at checkout) · Claim (convert to
  sale at payment) · Release (free on cancel/expire) - same words in code, SQL,
  docs. The atomic `UPDATE … WHERE (stock - stock_reserved) >= qty` guard makes
  concurrent checkouts safe; `RowsAffected() == 0` maps to a domain sentinel.
- **Ports & adapters.** `payment.Provider` and `shipping.Provider` are ports;
  `propay` and `TableProvider` are adapters; `Registry` selects by config. **Port
  only where ≥2 implementations are plausible** - otherwise a struct.
- **12-Factor.** All config via ENV (`internal/config`). Secrets never in code.
  `config.Load()` exits if `DATABASE_URL` or `JWT_SECRET` is missing.
- **Comments explain WHY, not WHAT.** Name things clearly; comment only the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bulletdev/bullet-commerce](https://github.com/Bulletdev/bullet-commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
