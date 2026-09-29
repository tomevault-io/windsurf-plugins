---
trigger: always_on
description: > **Map, not manual.** This file is the table of contents for the Livepeer Modules
---

# AGENTS.md

> **Map, not manual.** This file is the table of contents for the Livepeer Modules
> Suite. It points to the system of record in `docs/`. Keep it short (~100 lines).
> When in doubt, **link — don't inline.** Operating principles live in
> [`docs/design-docs/core-beliefs.md`](docs/design-docs/core-beliefs.md).

## What this repository is

The **Livepeer Modules Suite** is an umbrella (meta) repository. It aggregates the
individual Livepeer Module repositories as **git submodules** and provides a single,
high-level, agent-legible overview of the whole suite and how its parts fit together.

- This repo holds **documentation and submodule pointers** — not module source code.
  Each module's code lives in its own repository, mounted under `modules/<name>/`.
- Goal: an agent or human can understand the entire suite and the relationships
  between modules **directly from this repository**.
- The Livepeer Modules are the productized features that enable the Livepeer
  protocol. They are developed independently and released on the network's cadence;
  this suite tracks specific submodule revisions so the overview always corresponds
  to a known, reproducible set of releases.

## Two axes: capabilities and repos

- **Capabilities** (the "what") — the conceptual feature areas, one spec each in
  [`docs/product-specs/`](docs/product-specs/index.md).
- **Repos** (the "where") — the actual submodules, one doc each in
  [`docs/repos/`](docs/repos/index.md). **A single repo can implement several
  capabilities** (e.g. `livepeer-network-modules` covers most of the supply side).

New terms? Start with the [`docs/glossary.md`](docs/glossary.md).

## The capabilities (the suite)

One line each. Full specs in [`docs/product-specs/`](docs/product-specs/index.md).

| Capability | One-liner | Spec |
| --- | --- | --- |
| Gateways | Demand-side entry point: discover, pay, forward work | [gateways](docs/product-specs/gateways.md) |
| Orchestrators | Supply side: a workload-agnostic broker + daemons that serve paid work | [orchestrators](docs/product-specs/orchestrators.md) |
| Runners | The backends a broker dispatches to (the provided capability) | [runners](docs/product-specs/runners.md) |
| Pools | Control plane aggregating member backends behind one orch identity | [pools](docs/product-specs/pools.md) |
| Payment | Probabilistic micropayment tickets exchanged for work | [payment](docs/product-specs/payment.md) |
| Service Registry | On-chain pointer + off-chain signed manifest of capabilities | [service-registry](docs/product-specs/service-registry.md) |
| Discover | Resolver API gateways use to find and select orchestrators | [discover](docs/product-specs/discover.md) |
| Payment Clearinghouse | Settlement/distribution of payments (+ SDKs) | [payment-clearinghouse](docs/product-specs/payment-clearinghouse.md) |
| SDKs | Client/integration libraries spanning the suite | [sdks](docs/product-specs/sdks.md) |
| Reference Apps | Example gateway apps showing how to build on the suite | [reference-apps](docs/product-specs/reference-apps.md) |

### Observability & Reporting (off-network — tracks on-chain activity)

| Capability | One-liner | Spec |
| --- | --- | --- |
| Protocol Explorer | Indexes + prices + serves Livepeer on-chain activity (API + web explorer) | [protocol-explorer](docs/product-specs/protocol-explorer.md) |
| Network Bot | Reports payouts/activity to Discord by polling the explorer | [network-bot](docs/product-specs/network-bot.md) |

## Repositories (submodules)

| Repo | Implements | Doc |
| --- | --- | --- |
| livepeer-network-modules | Supply-side core: Orchestrators, Runners, Pools, Payment, Service Registry, Discover, Protocol, SDKs | [repos/livepeer-network-modules](docs/repos/livepeer-network-modules.md) |
| livepeer-open-clearinghouse | Demand-side control plane: Payment Clearinghouse, Payment issuance, SDKs, Discover proxy, gateway control-plane half. Consumes network-modules daemons. | [repos/livepeer-open-clearinghouse](docs/repos/livepeer-open-clearinghouse.md) |
| livepeer-modules-openai-runners | Runner backends: OpenAI/Cohere-shaped AI services (chat, embeddings, audio, TTS, image, rerank) that sit behind the capability broker. | [repos/livepeer-modules-openai-runners](docs/repos/livepeer-modules-openai-runners.md) |
| livepeer-modules-transcode-runners | Runner backends: video (VOD transcode, ABR ladder, live RTMP→HLS). live-runner = `live-session-gateway-ingest@v0` ("Option B"). | [repos/livepeer-modules-transcode-runners](docs/repos/livepeer-modules-transcode-runners.md) |
| livepeer-modules-transcode-gateway | Full in-path video Gateway (VOD ABR + live RTMP→HLS). Operator-funded via LOC; no local payer/resolver daemons. | [repos/livepeer-modules-transcode-gateway](docs/repos/livepeer-modules-transcode-gateway.md) |
| livepeer-modules-openai-gateway | Full in-path OpenAI-compatible AI Gateway ("change `base_url`"). TS/Fastify LOC-mediated twin of transcode-gateway; fronts the openai-runners. | [repos/livepeer-modules-openai-gateway](docs/repos/livepeer-modules-openai-gateway.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cloud-SPE/livepeer-modules-suite](https://github.com/Cloud-SPE/livepeer-modules-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
