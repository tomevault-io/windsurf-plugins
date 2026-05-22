---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Autentico is a self-contained OAuth 2.0 / OpenID Connect (OIDC) Identity Provider built with Go and SQLite. It implements the full authentication lifecycle: Authorization Code + PKCE, ROPC, Refresh Token grants, MFA (TOTP + email OTP), WebAuthn/passkeys, SSO sessions, trusted devices, token introspection/revocation, dynamic client registration, and an embedded React admin UI.

## Common Commands

```bash
# Build
make build                    # Build everything (admin UI + Go binary, statically linked)
make build-go                 # Build Go binary only (uses pre-built admin UI in pkg/admin/dist)
go build -o autentico main.go

# Run
make run                      # go run main.go start

# Test
make test                     # go test -p 1 -v ./...
go test ./pkg/token/...       # Run tests for a specific package
go test -run TestCreateUser ./pkg/user/...  # Run a single test

# Lint and format
make lint                     # Requires golangci-lint
make fmt                      # Format code with gofmt
go vet ./...                  # Static analysis

# Bootstrap a new .env (RSA key, CSRF secret, token signing secrets)
./autentico init              # or: ./autentico init --url https://auth.example.com

# Admin UI (frontend only, copies built assets into pkg/admin/dist)
make admin-ui-build

# Swagger documentation
make docs                     # Serves Swagger UI at localhost:8888
make generate-docs            # Regenerate swagger files from handler annotations
```

## Headless Onboarding & Admin Token (CI / Testing)

```bash
# 1. Remove existing DB for a clean start (if needed)
rm -f autentico.db

# 2. Start the server (auto-generates .env if missing)
./autentico start --auto-setup &
sleep 2

# 3. Create admin account with ROPC grant enabled (must run BEFORE start seeds the admin client,
#    or on a fresh DB — seedAdminClient skips if autentico-admin already exists)
./autentico onboard --username admin --password secret --email admin@test.com --enable-admin-password-grant

# 4. Get an admin bearer token via ROPC
TOKEN=$(curl -s http://localhost:9999/oauth2/token -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password&username=admin&password=secret&client_id=autentico-admin&scope=openid profile email" \
  | python3 -c "import json,sys; print(json.load(sys.stdin)['access_token'])")

# 5. Use the token for admin API calls
curl -s http://localhost:9999/admin/api/clients \
  -H "Authorization: Bearer $TOKEN"
```

**Important:** `onboard` must run before the first `start` seeds the admin client, or on a fresh DB. The `--enable-admin-password-grant` flag adds the `password` grant type to `autentico-admin`. If the client already exists without it, delete the DB and re-onboard.

## Playwright MCP (Browser Testing)

The Playwright MCP server provides browser automation tools (`mcp__playwright__*`) for testing UI flows.

**Setup:** The MCP server expects Chrome at `/opt/google/chrome/chrome`. If not installed, symlink the Playwright-managed Chromium:

```bash
# Install Playwright browsers (if not already cached)
npx playwright install chromium

# Symlink to the expected path (requires sudo)
sudo mkdir -p /opt/google/chrome
sudo ln -sf ~/.cache/ms-playwright/chromium-*/chrome-linux64/chrome /opt/google/chrome/chrome
```

**Debug UI as test client:** The `debug-ui/` directory contains a Vite+React app that acts as an OAuth2 client for manual testing. It runs on `http://localhost:5174` and uses client_id `autentico-debug`.

```bash
# Register the debug client (with consent_required for consent screen testing)
curl -s -X POST http://localhost:9999/admin/api/clients \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"client_id":"autentico-debug","client_name":"Debug UI","redirect_uris":["http://localhost:5174/callback"],"grant_types":["authorization_code","refresh_token","urn:ietf:params:oauth:grant-type:device_code"],"response_types":["code"],"scopes":"openid profile email offline_access","client_type":"public","token_endpoint_auth_method":"none","consent_required":true}'

# Add CORS origin for the debug UI
curl -s -X PUT http://localhost:9999/admin/api/settings \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"cors_allowed_origins": "http://localhost:5174"}'

# Start the debug UI dev server
cd debug-ui && pnpm dev
```

## Feature Development Workflow

When implementing new features, follow the checklist in `WORKFLOW.md`. Key points:

1. **Spec first** — read the relevant RFC/spec sections before writing code
2. **RFC annotations** — every return path and validation check must have an inline comment referencing the spec section
3. **Tests** — always add both positive and negative tests. Four layers exist:
   - **Unit** (`pkg/*/…_test.go`) — always required
   - **E2E** (`tests/e2e/`) — when the feature spans multiple endpoints
   - **Functional** (`tests/functional/`) — black-box HTTP tests against a running server
   - **Browser** (`tests/browser/`) — when the feature has a UI component (Playwright)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eugenioenko/autentico](https://github.com/eugenioenko/autentico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
