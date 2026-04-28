---
trigger: always_on
description: A statically-compiled Go binary that extracts CRD JSON schemas from a Kubernetes cluster and publishes them to a Cloudflare Pages website. Runs as a long-lived Deployment (watch mode) or CronJob in a K3s cluster. Deployed in a nonroot distroless container.
---

# CLAUDE.md - Project Context for crd-schema-publisher

## What This Is

A statically-compiled Go binary that extracts CRD JSON schemas from a Kubernetes cluster and publishes them to a Cloudflare Pages website. Runs as a long-lived Deployment (watch mode) or CronJob in a K3s cluster. Deployed in a nonroot distroless container.

## Repository Layout

```text
charts/         Helm chart (OCI-distributed via GHCR, cosign-signed)
cmd/            Entrypoint and subcommand dispatch (run/extract/upload/watch/preview)
converter/      OpenAPI v3 -> JSON Schema transforms (ported from openapi2jsonschema.py)
extractor/      client-go CRD listing, schema extraction, file writing, config builder
index/          HTML index generation (deepspace theme, client-side search, starfield/flare effects)
publisher/      Cloudflare Pages direct upload API client + BLAKE3 hashing
renderer/       HTML schema page renderer (collapsible property trees, type badges, constraints)
theme/          Shared CSS/HTML/JS constants (deepspace theme, starfield, flare, toggle, toast, footer)
metrics/        Prometheus metrics (stdlib-only, atomic counters/gauges, text exposition format)
watcher/        CRD informer watch loop, debounce, leader election, health server, metrics wiring
```

## Build & Test

```bash
# Run all tests
go test ./...

# Vet
go vet ./...

# Lint (requires golangci-lint)
golangci-lint run

# Enable pre-commit hook
git config core.hooksPath .githooks

# Cross-compile (matches CI targets)
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -ldflags="-s -w" -o crd-schema-publisher ./cmd/
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o crd-schema-publisher ./cmd/

# Local extract (requires kubeconfig)
KUBECTL_CONTEXT=my-context OUTPUT_DIR=./output go run ./cmd/ extract

# Preview index UI locally (no cluster needed)
go run ./cmd/ preview
```

## Architecture

### Subcommands

- `run` (default) — extract + upload
- `extract` — extract CRDs and write JSON schemas + index.html to OUTPUT_DIR
- `upload` — upload OUTPUT_DIR contents to Cloudflare Pages
- `watch` — long-lived process: informer watches CRDs, debounces events, runs extract+publish cycles. Leader election for multi-replica safety.
- `preview` — generate index with sample data (or real schemas via OUTPUT_DIR) and serve on localhost. No cluster or credentials needed. Handles signal cleanup of temp directories.

### Configuration (env vars)

| Var | Required | Default | Purpose |
| --- | -------- | ------- | ------- |
| `CLOUDFLARE_API_TOKEN` | Yes (run/upload) | — | CF API token |
| `CLOUDFLARE_ACCOUNT_ID` | Yes (run/upload) | — | CF account ID |
| `CF_PAGES_PROJECT` | No | `kubernetes-schemas` | CF Pages project name |
| `OUTPUT_DIR` | No | `/output` | Schema output directory |
| `KUBECTL_CONTEXT` | No | — | K8s context (local dev only) |
| `DEBOUNCE_SECONDS` | No | `15` | Seconds to wait after last CRD event before publishing (watch mode) |
| `POD_NAME` | Yes (watch) | — | Pod identity for leader election (set via downward API) |
| `POD_NAMESPACE` | Yes (watch) | — | Namespace for leader lease (set via downward API) |
| `LEASE_NAME` | No | `crd-schema-publisher` | Name of the Lease resource (watch mode) |
| `HEALTH_PORT` | No | `8080` | Port for liveness/readiness probes (watch mode) |
| `SKIP_RENDER` | No | — | Set to `true` to skip HTML schema page rendering |
| `BASE_PATH` | No | — | URL path prefix for subpath deployments (e.g., `/iac` for GitHub Pages) |
| `PREVIEW_ADDR` | No | `127.0.0.1:8989` | Listen address (preview mode) |

### Key Design Decisions

- **No CGO.** Binary is statically linked. BLAKE3 uses `github.com/zeebo/blake3` (pure Go).
- **Cloudflare Pages direct upload API** is undocumented. Implementation reverse-engineered from wrangler source (`cloudflare/workers-sdk`). The upload flow uses JWT auth for asset operations and API token auth for deployment creation. See `publisher/publisher.go` for the full 6-step flow.
- **BLAKE3 file hashing** exactly matches wrangler's `hashFile`: `hex(blake3(base64(content) + extension))[0:32]`. Do not change this algorithm without verifying against wrangler source.
- **OpenAPI v3 to JSON Schema conversion** is an improved port of `openapi2jsonschema.py` from datreeio/CRDs-catalog (via yannh/kubeconform). Three transforms applied in order: additionalProperties, replaceIntOrString, allowNullOptionalFields. Known divergences from the Python original, all intentional improvements for kubeconform/IDE validation correctness: (1) nullable applies only to fields *not* in the required list — Python disables nullable for *all* siblings when any sibling is required; (2) `replaceIntOrString` preserves existing keys alongside oneOf — Python discards the entire dict; (3) root object and array items are not made nullable — Python makes them nullable unnecessarily. A golden E2E test (`extractor/testdata/golden_certificate_v1.json`) freezes the converter output and catches any regression.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sholdee/crd-schema-publisher](https://github.com/sholdee/crd-schema-publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
