---
trigger: always_on
description: Machine-readable rules for agents working in this repo. Human overview lives in
---

# AGENTS.md — freebucks-proxy operating guide

Machine-readable rules for agents working in this repo. Human overview lives in
`README.md`; visual grammar in `DESIGN.md`; architecture decisions in
`docs/decisions/`; multi-agent workflow in the repo's local dev notes
(`devdocs/`, gitignored — present only in a development checkout).

## 1. Identity

- Go 1.26 (`go.mod`) gateway for the upstream wire protocol. OpenAI-compatible surfaces
  (`/v1/chat/completions`, `/v1/models` — see `backend/cmd/freebucks-proxy/e2e_test.go`,
  `backend/internal/cli/cli_serve.go`) plus an Anthropic translation layer
  (`backend/internal/server/anthropic*.go`).
- Svelte 5 dashboard (`frontend/`, `freebucks-proxy-dashboard`) embedded via
  `go:embed` (`backend/internal/dashboard/assets_embed.go`) and served at `/admin`.
  Health probe: `GET /healthz` → 200.
- Modes (`backend/internal/config/config.go:HybridBridgeMode/EffectiveMode`):
  pooled (`AUTH_TOKENS` set + `BRIDGE_ENABLED=0`), bridge (`AUTH_TOKENS`
  empty, per-request client token), hybrid (default when `AUTH_TOKENS` set:
  `API_KEYS` credential uses the pool, any other credential relays as bridge).
- Upstream credits meter (wire fields: `freebucks*`): the wire `prices` map is
  the sole cost source; charge-once at session start; 1h sessions; `DELETE`
  refund. The daily refill follows the reset the session payload advertises
  (`resetTimeZone`/`resetAt` — the account's own local midnight; Pacific
  midnight only on servers that omit the zone), while the proxy's own counters
  bucket on the Pacific day (`pool/spend.go:bucketStart`).
  `deepseek/deepseek-v4-flash` bills 15 Freebucks/hour with a server-sent
  off-peak offer (vendor fixture: 10 in the 22:00-06:00 UTC window,
  `common/src/util/__tests__/freebuff-off-peak-price.test.ts:4-9`), and its
  peak PRICE card still doubles even though the peak-serving pause was
  removed (`common/src/constants/freebuff-models.ts:1406-1409` — the vendor
  comment is explicit that only the pause went away). Daily allowances differ
  per country group (`common/src/constants/freebuff-countries.ts:19-62`; the
  amounts themselves are server-side). The wire `prices`/`offPeak` maps —
  never this note — are the source of truth.

## 2. Topology

- `backend/` — Go gateway (`cmd/`, `internal/`). `internal/` packages include
  `server`, `pool`, `upstream`, `session`, `store`, `config`, `dashboard`,
  `modelcat`, `registry`, `wirefacts`, `convert`.
- Client tool-name layer: `backend/internal/convert` renames foreign client
  tool names to official signature names on the request leg and restores the
  client's own names on every response path, including the name-uniqueness
  virtualization (`mcp__*`) and the wire-grammar legalization
  (`^[A-Za-z0-9_-]{1,64}$`). Rules, rejected alternatives and invariants:
  `docs/decisions/tool-name-translation.md`.
- `frontend/` — Svelte 5 SPA. Committed bundle
  `backend/internal/dashboard/dist` is what the binary serves.
- The gitignored upstream vendor clone (live checkout) — never commit; the exact path lives in `scripts/check-upstream.sh`. Source of truth for all wire/registry/model work. Keep freshly fetched to `origin/main` before starting; pins live in `backend/internal/wirefacts/testdata/wire/snapshots.json` (`upstream_sha`) + `scripts/vendor-version.txt`, verified by `scripts/check-upstream.sh`.
- `scripts/` — `sync-upstream.sh`, `check-upstream.sh` (canonical parity check),
  `review-wire-drift.sh`, `drift-exact.sh` (exact export-level MODEL/PRICE/WIRE report),
  `drift-tui.sh` (picker wireframe per tier: rows/order/fields + refresh checklist).
- `.github/workflows/` — `ci.yml` (jobs `test`, `frontend`), `lint.yml` (job
  `golangci`), `codeql.yml` (job `analyze`), `dependency-review.yml`,
  `upstream-drift.yml`, `release.yml`.

## 3. Commands

### Fast Audit by Domain (< 5 seconds)

```sh
# Frontend Dashboard: typecheck in ~3s (zero screenshot / e2e overhead)
npm --prefix frontend run check
# Targeted single-spec Playwright test (when validating a specific UI flow)
npx --prefix frontend playwright test e2e/<target>.spec.ts

# Backend Gateway: test specific package without full-suite 10m race runner
go test -v ./backend/internal/<pkg>/...    # e.g. ./backend/internal/pool/...
go vet ./backend/internal/<pkg>/...

# Upstream Parity / Wire Drift: instant export-level check
bash scripts/drift-exact.sh

# Client tool-name corpus: regenerate the fixture from the gitignored
# reference/ corpus, then run the sweep (wire-unique + grammar-legal +
# exact client-name restore per harness)
bash scripts/extract-tool-calls.sh && go test ./backend/internal/convert/ -run Corpus -v

# Config validation
go test ./backend/internal/config/...

# Test tiers (seconds → minutes; CI stays authority for -race/golangci/e2e)
task verify:quick                            # gofmt + vet + svelte-check, seconds
task test:fast                               # all backend pkgs except pool+server+cmd
go test -short ./backend/internal/<pkg>/...  # skip heavy keepers (pool ladder/pin, conformance, mock-e2e)
task test:pool | task test:server | task test:e2e   # heavy lanes, run alone serially
npm --prefix frontend run test:unit          # node unit suites, no browser
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trefeon/freebuff-proxy](https://github.com/trefeon/freebuff-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
