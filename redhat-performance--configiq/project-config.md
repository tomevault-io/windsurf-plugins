---
trigger: always_on
description: This file gives coding agents context about this project.
---

# AGENTS.md — ConfigIQ

This file gives coding agents context about this project.
Read this before making any changes.

## What this project is

ConfigIQ is a web application for LLM inference sizing, GPU comparison, and
cost modeling. It is deployed at configiq.dev and uses the AIConfigurator
REST API (server URL set via `AICONFIGURATOR_GATEWAY_URL`, e.g.
`https://aiconfigurator.dev`) for GPU recommendations and memory estimation.

## Tech stack

- **Framework**: Next.js 14 App Router + TypeScript
- **UI**: PatternFly v5 (Red Hat's design system) — no Tailwind, no shadcn
- **Charts**: PatternFly Victory Charts
- **Fonts**: Red Hat Display (headings, metrics), Red Hat Text (body), Red Hat Mono (numbers, code, labels)
- **Deployment**: Vercel

## Documentation

**Essential reading:**

- **[docs/DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md)** - Complete design system
  - Typography scale and font usage rules
  - Color palette and contrast requirements  
  - Spacing scale and layout patterns
  - Reusable component patterns (flip tiles, search, accordions, tour)
  - Animation guidelines and accessibility requirements

- **[docs/ARCHITECTURE_DETAILED.md](docs/ARCHITECTURE_DETAILED.md)** - System architecture
  - Component diagrams showing module relationships
  - Data flow visualizations
  - Inference engine internal architecture
  - API architecture and type system
  - Current integration status and roadmap

**Critical**: Follow DESIGN_SYSTEM.md for UI work and ARCHITECTURE_DETAILED.md for understanding how components interact.

## Project structure

```text
app/                    # Next.js App Router pages
  layout.tsx            # Root layout with AppShell
  page.tsx              # Homepage
  performance/          # Performance tool
  recommend/            # Recommend sizing tool
  kv-cache/             # KV Cache Calculator tool
  gpu-explorer/         # GPU Explorer tool
  hybrid-savings/       # Hybrid Savings tool
  routing/              # Routing Economics tool
  settings/             # App settings
  api/                  # Next.js API routes — proxy to AIConfigurator REST API
    recommend/          # POST — GPU sizing
    estimate/           # POST — GPU performance
    memory/             # POST — memory breakdown
    gpus/               # GET — GPU catalog + optional live pricing
    hf-config/          # GET — Hugging Face model config lookup
    health/             # GET — health check
components/
  layout/
    AppShell.tsx        # Page shell with sidebar nav
lib/
  gpu-math/             # Legacy client-side GPU sizing — historical/fallback only
  api/                  # AIConfigurator API clients — source of truth for GPU math
  pricing/              # Cloud GPU pricing data
  hooks/                # useAicCatalog → /api/catalog, useCostings →
                        # /api/costings/* — both go through same-origin api/
                        # proxies (server-side gateway env), no direct external
                        # fetches from the browser
  utils/
    format.ts           # Number/unit formatting helpers
docs/                   # Architecture docs and ADRs
public/                 # Static assets
services/               # Backend Python microservices (FastAPI), built as
                        # their own container images, released in unison with
                        # the frontend (see .github/workflows/build.yml)
  configiq-py/          # Shared Python library (import name: configiq):
                        # GPU systems catalog, OpenTelemetry + MCP wiring.
                        # Consumed by the services below as a uv path dependency.
  aiconfigurator/       # Thin FastAPI wrapper over the aiconfigurator SDK
                        # (GPU sizing, performance + memory estimation)
  aicostings/           # GPU + LLM pricing API (scrapes providers into Valkey)
```

The `services/` tree is Python (FastAPI + uv), separate from the Next.js
frontend at the repo root. GPU math still lives in the aiconfigurator SDK — the
`aiconfigurator` service is only a REST wrapper. See each service's `README.md`
for details.

## Critical rules

1. **GPU math belongs in AIConfigurator** — never write sizing formulas inside
   React components. Components call `lib/api/` clients, which talk to the
   AIConfigurator REST API via the `app/api/` proxy routes. `lib/gpu-math/` is
   legacy client-side sizing kept for historical/fallback use only — do not
   add new formulas there.

2. **PatternFly only for UI** — do not install or use Tailwind, shadcn/ui,
   Material UI, or any other component library. PatternFly is the single
   source of truth for components and styling.

3. **App Router patterns** — use server components by default. Add `"use client"`
   only when you need browser APIs, state, or event handlers.

4. **TypeScript strict** — no `any` types. Define proper interfaces in the
   relevant lib file or component.

5. **Sentence case everywhere** — Red Hat brand standard. No title case in
   headings or labels.

6. **Follow the design system** — see [docs/DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md) for typography, colors, spacing, components, and patterns. Don't invent new styles or components without checking the design system first.

## Commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-performance/configiq](https://github.com/redhat-performance/configiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
