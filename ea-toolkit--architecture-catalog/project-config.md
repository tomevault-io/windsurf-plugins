---
trigger: always_on
description: A lightweight, Git-native architecture catalog. It replaces monolithic tools like Archi with a simple repo structure: architecture elements are registered as Markdown files, diagrams are draw.io files, and Python scripts validate, extract, refresh, and generate dashboards.
---

# CLAUDE.md

## What is this project?

A lightweight, Git-native architecture catalog. It replaces monolithic tools like Archi with a simple repo structure: architecture elements are registered as Markdown files, diagrams are draw.io files, and Python scripts validate, extract, refresh, and generate dashboards.

The goal is to be an open-source, lightweight alternative for architects who want Git-friendly, AI-readable architecture models without learning a DSL or paying for enterprise tooling.

## Architecture (how it works)

```
registry-mapping.yaml  ──→  registry-loader.ts  ──→  Astro pages
  (schema: types,             (reads .md files,        (static HTML
   fields, relationships)      resolves refs,           at build time)
                               builds graph)
```

**Single source of truth:** `models/registry-mapping.yaml` drives everything — the UI, validation, and data loading. Adding a new element type = add one entry to this YAML file + create a `_template.md`. Zero code changes.

**Data pipeline:** registry-mapping.yaml → `registry-loader.ts` (loadRegistry()) → `registry.ts` (bridge) → Astro pages. The loader builds an in-memory graph with elements, edges, and indexes (byType, byDomain, byLayer).

**Key design:** Framework-agnostic. ArchiMate alignment is optional, not required. Works with any architecture vocabulary. Layers are flat (1 to N), each with sub-folders for element types. No nested child layers.

## Current State

### Registry (registry-v2/)
- **4-layer structure** — flat layers, each with typed sub-folders
- **Template-driven** — each element type has a `_template.md` with typed YAML frontmatter
- **Example domains**: "Customer Management", "Billing & Payments", "Analytics & Insights" — 3 fictional B2B SaaS CRM domains with 71 elements across all 4 layers
- **Health:** 71 healthy, 71 connected, 0 orphans, 0 broken refs, 84 pages

### Catalog UI (catalog-ui/)
- **Astro 5 + React** static site for browsing the registry
- **Schema-driven** — all types, layers, and relationships derived from `models/registry-mapping.yaml`
- **ReactFlow graphs** for visualizing element relationships (via `@xyflow/react` + dagre)
- **Diagram viewer** supporting PlantUML, BPMN, and draw.io formats
- **CSS pattern:** EventCatalog-inspired RGB variable pattern: `--ec-page-bg: 255 255 255` used as `rgb(var(--ec-page-bg))`
- **3-panel layout:** icon bar (56px) + nested sidebar (315px) + main content

### Piece 1 / Piece 2 Strategy
- **Piece 1 (this repo):** registry-mapping.yaml → registry skeleton → UI. Ships first.
- **Piece 2 (future):** meta model → registry-mapping.yaml generation. Convenience tool, not critical path.

---

## Skills (Slash Commands)

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/enterprise-platform-archi` | `/enterprise-platform-archi [question]` | Enterprise Platform domain Q&A, create entries, proposals |
| `/validate` | `/validate` | Run model validation, show errors and orphans |
| `/dashboard` | `/dashboard` | Generate HTML health dashboard |
| `/new-entry` | `/new-entry [type] [name]` | Create registry entry (guided wizard) |
| `/scaffold-component` | `/scaffold-component [Name]` | Scaffold React component + test file |
| `/deploy` | `/deploy [--dry-run] [--target catalog\|docs\|all]` | Build & deploy to Firebase Hosting |
| `/crawl-apis` | `/crawl-apis <path> [--domain <name>] [--write]` | Scan codebase for APIs, propose registry entries |
| `/crawl-data` | `/crawl-data <path> [--domain <name>] [--write]` | Scan codebase for data models, propose registry entries |

### Examples
```
/enterprise-platform-archi What data does Tenant Management own?
/enterprise-platform-archi Create a registry entry for Payment Gateway
/validate
/dashboard
/new-entry data-object "Payment Record"
/scaffold-component CapabilityHeatmap
/deploy --dry-run
/deploy --target catalog
/crawl-apis ./src --domain "Customer Management"
/crawl-data ./src --domain "Billing & Payments"
```

### Why Skills?
- **Explicit invocation** — Type `/enterprise-platform-archi` to guarantee the right context
- **Scoped search** — Each domain skill searches its own files first, minimizing token usage
- **Pattern for teams** — `/<domain>-archi` is learnable: `/payments-archi`, `/logistics-archi`, etc.

---

## Repo Structure

```
.claude/
  skills/
    enterprise-platform-archi/SKILL.md  # /enterprise-platform-archi skill
    validate/SKILL.md               # /validate skill
    dashboard/SKILL.md              # /dashboard skill
    new-entry/SKILL.md              # /new-entry skill
    scaffold-component/SKILL.md     # /scaffold-component skill
  agents/
    domain-expert.md                # Read-only domain Q&A (auto-delegation)
    fe-developer.md                 # Frontend developer (catalog-ui/)
    registry-agent.md               # Registry data specialist (registry-v2/, views/)
    test-writer.md                  # Test specialist (pytest + vitest)
  hooks/
    welcome.sh                      # Shows skills on session start

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ea-toolkit/architecture-catalog](https://github.com/ea-toolkit/architecture-catalog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
