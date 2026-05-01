---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

Radar is a modern Kubernetes visibility tool — local-first, no account required, no cloud dependency, fast. It provides topology visualization, event timeline, service traffic maps, resource browsing, Helm management, and cluster audit (best-practices scanning). Runs as a kubectl plugin (`kubectl-radar`) or standalone binary and opens a web UI in the browser. Open source, free forever. Built by Skyhook.

## Reference Docs — MUST READ before making changes

Not everything is in this file. The following files contain critical details that are **not duplicated here**. You MUST read them when working in the relevant area — do not guess or rely on memory.

| When you are... | Read this file FIRST |
|-----------------|---------------------|
| Adding or modifying **HTTP endpoints** | `internal/server/server.go` — all routes are defined here |
| Adding or modifying **CLI flags** | `cmd/explorer/main.go` — flag definitions and defaults |
| Adding a **new CRD integration** (renderer, topology, discovery) | [docs/INTEGRATION_GUIDE.md](docs/INTEGRATION_GUIDE.md) — full checklist with collision gotchas |
| Working on **resource renderers** | `packages/k8s-ui/src/components/resources/renderers/` — all existing renderers live here |
| Understanding **cluster connection behavior** | [docs/configuration.md](docs/configuration.md) — kubeconfig precedence, multi-context, in-cluster |
| Working on **MCP tools or AI context** | [docs/mcp.md](docs/mcp.md) + `internal/mcp/tools.go` — tool definitions and design rationale |
| Writing or modifying **frontend UI / styling** | [DESIGN.md](DESIGN.md) — theme tokens, do's/don'ts, component patterns |
| Touching anything library consumers import | `web/package.json` + `web/src/index.ts` — `web/` IS the `@skyhook-io/radar-app` npm package. Public surface: `RadarApp`, runtime-config setters (`setApiBase` etc.), `NavCustomization`. Breaking it breaks all downstream consumers. |
| Adding or changing **api/fetch call sites** | `web/src/api/config.ts` — all fetches go through `getApiBase()`, `apiUrl()`, `getWsUrl()`, `getAuthHeaders()`, `getCredentialsMode()`. New fetch sites must use these helpers so library consumers (Radar Hub) can override per-cluster. |
| Embedding Radar inside another app | `web/src/RadarApp.tsx` + `web/src/context/NavCustomization.tsx` — `apiBase`, `basename`, `router`, `navSlots` props. Changes to this API surface are breaking. |

## Library distribution

In addition to the standalone binary, Radar's frontend is published as **`@skyhook-io/radar-app`** (source-only npm package, same model as `@skyhook-io/k8s-ui`). The `web/` directory IS the package: `web/package.json` carries the npm metadata, `web/src/index.ts` is the library entry, and Radar's own binary entry (`web/src/main.tsx`) consumes the same source.

Publish with tag `radar-app-v<semver>` — see `.github/workflows/publish-radar-app.yml`.

Consumers get:
- `<RadarApp apiBase basename router navSlots queryClient />` — the whole app as one component
- Runtime config setters for cross-cutting behavior (`setApiBase`, `setBasename`, `setAuthHeadersProvider`, `setCredentialsMode`) for non-React code paths
- `NavCustomization` type for nav slot injection

Known consumers: Radar Hub (`skyhook-dev/radar-hub-web`).

**Backwards-compat rule:** adding props is fine; removing or renaming `apiBase` / `basename` / `navSlots` fields is breaking. Bump major version.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         User's Machine                          │
│                                                                 │
│   ┌─────────────────┐                   ┌───────────────────┐  │
│   │    Browser      │◄── HTTP/SSE/WS ──►│  Radar Binary     │  │
│   │  (React + UI)   │                   │  (Go + Embedded)  │  │
│   └─────────────────┘                   └───────┬───────────┘  │
│                                                  │              │
│   ┌─────────────────┐                            │              │
│   │   AI Tools      │◄──── MCP (HTTP) ───────────┤              │
│   │  (Claude, etc.) │                            │              │
│   └─────────────────┘                            │              │
│                                                  │              │
└──────────────────────────────────────────────────│──────────────┘
                                                   │
                                         ┌─────────┴─────────┐
                                         │  kubeconfig       │
                                         │  (~/.kube/config) │
                                         └─────────┬─────────┘
                                                   │
                                         ┌─────────┴─────────┐
                                         │  Kubernetes API   │
                                         │  (direct access)  │
                                         └───────────────────┘
```

## Project Structure

```
radar/
├── cmd/
│   ├── explorer/              # CLI entry point (main.go)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skyhook-io/radar](https://github.com/skyhook-io/radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
