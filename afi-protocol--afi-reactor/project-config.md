---
trigger: always_on
description: **afi-reactor** is AFI Protocol's **scored-signal evaluation runtime (District One)**.
---

# afi-reactor — Agent Instructions

**afi-reactor** is AFI Protocol's **scored-signal evaluation runtime (District One)**.
It ingests signals, runs **one** manifest-driven graph executor over five
provider-backed enrichment lanes, produces a UWR score via the Froggy analyst,
and hands the scored signal to District Two evidence/provenance. It **stops at
scored** — validator certification and trade execution are downstream / external.

**Naming Note**: Use "afi-reactor" naming throughout.

**Global Authority**: All agents/droids operating in AFI Protocol repos must
follow `afi-config/codex/governance/droids/AFI_DROID_CHARTER.v0.1.md` (LIVE
authority). If this AGENTS.md conflicts with the Charter, **the Charter wins**.

For global droid behavior and terminology, see:
- `afi-config/codex/governance/droids/AFI_DROID_CHARTER.v0.1.md`
- `afi-config/codex/governance/droids/AFI_DROID_PLAYBOOK.v0.1.md`
- `afi-config/codex/governance/droids/AFI_DROID_GLOSSARY.md`

---

## Architecture (current)

Pipeline composition is **data, not code**: there is no hardcoded pipeline in
source. Flow:

1. **HTTP ingress** — Universal Signal Schema v1.1 via the Gateway signals path
   (reactor endpoint `POST /api/webhooks/tradingview`; the governed
   structured-ingress front is afi-gateway `POST /api/v1/signals`), plus the
   internal CPJ adapter (`POST /api/ingest/cpj`).
2. **AJV validation** — `src/uss` (USS) and `src/cpj` (CPJ) validators.
3. **One manifest-driven GraphExecutor** — `src/pipeline/executor.ts`,
   constructed once in `src/config/runtimeComposition.ts`, running the
   registered graph **`froggy-trend-pullback` v1.1.0**.
4. **Five vendor-neutral, provider-backed enrichment lanes** — technical,
   pattern, sentiment, news, aiMl. Each lane selects an explicit
   **ProviderInstance** from the governed afi-config registries
   (`registries/{providers,provider-instances,provider-bindings,pipelines,analysis-plugins,analyst-strategies}`).
   The static adapter registry in **`src/providers/`** (`adapterRegistry.ts` +
   `adapters/`) is the SOLE enrichment execution seam.
5. **Join → Froggy analyst → UWR score** — the five lane results join, then the
   Froggy analyst scores `trend_pullback_v1` from **afi-core**, producing a UWR
   score (Utility / Work-quality / Rarity; axes: structure, execution, risk,
   insight).
6. **District Two evidence/provenance** — the governed record
   `afi.scored-signal-evidence.v3` is persisted through **afi-infra** (the SOLE
   evidence writer) into MongoDB collection `scored_signal_evidence`.

**Output**: a scored-only signal with
`analystScore { uwrScore, uwrAxes { structure, execution, risk, insight } }`
(plus `rawUss`, optional `lenses`, optional `_priceFeedMetadata`, `scoredAt`,
`decayParams`, `meta`). There is **no** `validatorDecision` and **no** execution
block.

**Not the reactor's responsibility** (stops at scored): validator certification
and trade execution are downstream / external; mint orchestration lives in
`afi-mint`. The legacy Froggy demo personas (Alpha Scout / Pixel Rick / Val Dook
/ Execution Sim) were removed and must not return.

Scoring, UWR, decay, and evidence are unchanged and **byte-stable** (oracle
goldens).

---

## Build & Test

```bash
npm install              # install deps (siblings afi-core/afi-math/afi-config linked via file:../)
npm run build            # tsc → dist
npm run typecheck        # tsc --noEmit
npm test                 # jest: unit + guardrails + behavioral oracle goldens
npm run start:demo       # node dist/src/server.js (port 8080)
npm run esm:check        # ESM invariant lint
```

Real-Mongo proofs (gated; CI runs them):

```bash
npm run test:integration:unavailable   # 503 smoke when the store is unavailable
npm run test:integration:mongo         # evidence persistence
npm run test:integration:shutdown      # graceful shutdown
npm run test:oracle:mongo              # oracle byte-equivalence against real Mongo
```

Never regenerate oracle goldens to make a change pass — byte drift is the signal
that behavior changed.

---

## Graph Executor

The single execution path is the generic graph executor; composition is loaded
and validated at boot:

- `src/pipeline/registryLoader.ts` — loads + AJV-validates the governed
  registries (pipelines, analysis plugins, analyst strategies, provider
  bindings/instances) and verifies canonical `afi.hash.v1` pins at boot
  (fail-closed; the server refuses to boot on an invalid registry).
- `src/config/runtimeComposition.ts` — the boot-validated composition seam
  (`initRuntimeComposition`); tests may inject overlay registry roots.
- `src/config/strategyResolution.ts` — resolves the provider binding to a
  registered strategy triple (never free text).
- `src/pipeline/executor.ts` — executes the REGISTERED pipeline manifest
  (categories, ports, join determinism, failure policies) over the plugin
  registry (`src/pipeline/pluginRegistry.ts`).
- `src/providers/` — the provider-adapter framework; `adapterRegistry.ts` +
  `adapters/` is the SOLE enrichment execution seam (secrets only via the
  injected `SecretResolver`; never a key in a URL; output validated against the
  governed category contract).
- `src/evidence/` — evidence construction + District-2 provenance law

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AFI-Protocol/afi-reactor](https://github.com/AFI-Protocol/afi-reactor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
