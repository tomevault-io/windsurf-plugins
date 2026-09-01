---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

OCULTAR is a fully open-source, zero-egress PII detection and redaction proxy. It sits between a client and an upstream API (e.g., OpenAI, Gemini), intercepts requests, tokenizes all PII in-place using deterministic SHA-256 tokens (e.g., `[EMAIL_9c8f7a1b]`), and stores encrypted ciphertext in a local Vault. Responses can optionally rehydrate tokens back to plaintext for authorized callers. No raw PII ever reaches the upstream.

All components — including the Sombra agentic gateway — are open source. There is no enterprise/community split.

## Prerequisites

- **Go 1.25+** (workspace uses `go 1.25.8`; tested on `go1.26.3`)
- **CGO enabled** — DuckDB and libphonenumber require a C compiler (`gcc`/`clang`)
- **Node 18+** for the frontend apps

```bash
# Verify CGO is available
gcc --version
CGO_ENABLED=1 go build ./...
```

## Commands

```bash
# Build all Go modules (requires CGO)
make build

# Run all tests
make test

# Full workflow: sync workspace → provision model → build → test
# WARNING: `make all` provisions the SLM model — do not run in CI
make all

# Start the proxy (default port 8081)
go run ./apps/proxy

# Start the refinery HTTP server (port 8080)
go run ./services/refinery/cmd/main.go --serve 8080

# Start the SLM sidecar (local AI NER, port 8085)
go run ./apps/slm-engine/main.go

# Start the Sombra Gateway (port 8086)
go run ./apps/sombra

# Frontend (apps/dashboard or apps/web)
npm run dev
npm run build
```

Running a specific Go test:
```bash
cd services/refinery && CGO_ENABLED=1 go test ./... -run TestName
```

## Testing Conventions

All refinery detection logic must have table-driven tests with `input`/`wantRedacted` pairs. Use in-memory DuckDB (pass `""` as vault path) so tests are self-contained and don't leave `.db` files.

```go
func TestMyRule(t *testing.T) {
    v, _ := vault.New(config.Settings{VaultBackend: "duckdb"}, "")
    t.Cleanup(func() { v.Close() })
    config.InitDefaults()
    eng := refinery.NewRefinery(v, []byte("01234567890123456789012345678901"))

    cases := []struct {
        input string
        want  string // expected token type prefix, e.g. "[EMAIL_"
    }{
        {"send to alice@example.com", "[EMAIL_"},
    }
    for _, c := range cases {
        result, _ := eng.RefineString(c.input, "", nil)
        if !strings.Contains(result, c.want) {
            t.Errorf("input %q: want token %q in output, got %q", c.input, c.want, result)
        }
    }
}
```

**TestMain pattern** — any test package that calls `config.InitDefaults()` needs a `testmain_test.go` that `os.Chdir`s to the repo root so `configs/` relative paths resolve. See `services/refinery/pkg/proxy/testmain_test.go` as the reference.

**Entity Registry tests** — use `vault.Provider.RegisterEntity` to pre-seed canonical entities, then assert that all name variants collapse to a single `[TYPE_N]` token. See `services/refinery/pkg/refinery/refinery_entity_test.go`.

## Commit Style

Follow conventional commits with a scope matching the affected module:

```
feat(refinery): add FR_IBAN regex to Tier 1 rule engine
fix(vault): handle concurrent RegisterEntity race on DuckDB
docs: update ARCHITECTURE.md for v2.5
```

Common scopes: `proxy`, `refinery`, `vault`, `sombra`, `slm-engine`, `pii`, `gateway`, `docs`.

## Required Environment Variables

OCULTAR primarily uses **Doppler** for secret management. If running manually, copy `.env.example` to `.env`:

| Variable | Purpose |
|---|---|
| `OCU_MASTER_KEY` | 32-byte AES key for HKDF key derivation |
| `OCU_SALT` | Per-deployment salt |
| `OCU_PROXY_TARGET` | Upstream API base URL |
| `OCU_PROXY_PORT` | Proxy listen port (default `8081`) |
| `SLM_SIDECAR_URL` | SLM sidecar endpoint (default `http://localhost:8085`) |
| `SLM_ADAPTER` | Sidecar protocol: `privacy-filter` (default) or `openai-chat` (llama.cpp/Qwen) |
| `OCU_JWT_SECRET` | HS256 secret for Sombra JWT Bearer validation (generate: `openssl rand -hex 32`). If unset, Sombra is in insecure dev mode. |
| `OCU_AUDIT_PRIVATE_KEY` | Hex-encoded 32-byte Ed25519 seed for immutable audit log (generate: `openssl rand -hex 32`) |
| `OCU_AUDIT_LOG_PATH` | Audit log file path (default: `audit.log` alongside vault file) |
| `SLACK_SIGNING_SECRET` | Slack app signing secret for HMAC-SHA256 verification of incoming Slack events. Required when using the Slack connector; if unset, Sombra rejects all Slack webhook requests with HTTP 500. |

**Deprecated:** `TIER2_ENGINE` is a legacy alias for `SLM_ADAPTER` — the server logs `[DEPRECATED]` on startup if it is set. Use `SLM_ADAPTER` instead.

**Port note:** `apps/slm-engine` defaults to `:8085`; the refinery's `SLM_SIDECAR_URL` defaults to `http://localhost:8085`. Both services connect out of the box with no extra configuration.

**SLM sidecar (`apps/slm-engine`) — additional vars:**

| Variable | Purpose |
|---|---|
| `PYTHON_SIDECAR_URL` | URL of the Python privacy-filter process (default `http://localhost:8086`) |
| `PRIVACY_FILTER_MODEL_PATH` | HuggingFace model ID or local path (default `openai/privacy-filter`) |

## Architecture

### Go Workspace


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Edu963/ocultar](https://github.com/Edu963/ocultar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
