---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**make-skills** provides expert skills for designing, building, and deploying Make automation scenarios. Distributed as both a Claude Code plugin and as Open Agent Skills (compatible with 40+ AI agents via `npx skills add integromat/make-skills`). Published by Make under MIT license.

The skills connect to the remote Make MCP server:

- **`make`** — Make's hosted MCP server at `https://mcp.make.com`. Provides tools for app discovery, module configuration, connections, webhooks, data stores, and scenario lifecycle. Authenticated via OAuth (default) or MCP token.

## Repository Structure

```
.claude-plugin/
  plugin.json              # Plugin manifest (name, version, description)
  marketplace.json         # Marketplace metadata
.mcp.json                  # MCP server configuration (remote Make server)
skills/
  make-api-shell-connection-workflow/  # API-call shell provisioning + retrieval transport (4 reference files)
    SKILL.md
    discovery-and-shells.md
    connection-requests.md
    retrieval-execution.md
    sanitization-and-sharing.md
    examples/generic-api-shell-blueprint.json
  make-mcp-reference/      # MCP config & troubleshooting (1 reference file)
    SKILL.md
    references/transport-details.md
  make-module-configuring/  # Module configuration workflow (11 reference files)
    SKILL.md
    general-principles.md, connections.md, mapping.md, webhooks.md,
    data-stores.md, data-structures.md, keys.md, filtering.md,
    iml-expressions.md, aggregators.md, ai-agents.md
  make-scenario-building/   # Scenario design methodology (18 reference files)
    SKILL.md
    blueprint-construction.md, connections.md, webhooks.md,
    scheduling-and-triggers.md, routing.md, branching.md, merging.md,
    filtering.md, iterations.md, aggregations.md, mapping.md,
    error-handling.md, data-stores.md, subscenarios.md, bundles.md,
    ai-agents.md, quick-patterns.md, CONTRIBUTING.md
```

## Skills

Four auto-activated skills guide scenario building end-to-end. They divide responsibilities:

- **make-api-shell-connection-workflow** handles reusable Make API-call shell provisioning and Make-first SaaS retrieval routing
- **make-scenario-building** decides WHICH modules to use and WHY (scenario architecture)
- **make-module-configuring** handles HOW to configure each module (parameters, connections, mapping)
- **make-mcp-reference** covers MCP infrastructure (connection methods, scopes, troubleshooting)

### make-api-shell-connection-workflow

Reusable API-call shell workflow: provider/app resolution, connection reuse vs credential request, shell creation/patching, explicit interface setup, run validation, and SaaS retrieval via the shell contract.

References: 4 files (discovery-and-shells, connection-requests, retrieval-execution, sanitization-and-sharing) plus 1 example blueprint

### make-mcp-reference

MCP server configuration, OAuth vs token auth, scopes, troubleshooting connection issues. Activated when users ask about MCP setup, tokens, OAuth, or connection errors.

Reference: `references/transport-details.md`

### make-module-configuring

5-phase module configuration workflow: read interface (`app-module_get`), resolve RPCs, fill parameters, validate (`validate_module_configuration`), get app docs. Covers connections, mapping, webhooks, data stores, data structures, keys, filtering, IML expressions, and aggregators.

References: 11 files (general-principles, connections, mapping, webhooks, data-stores, data-structures, keys, filtering, iml-expressions, aggregators, ai-agents)

### make-scenario-building

Scenario design methodology: understand business need, discover apps/modules, select module composition, construct blueprint, deploy. Covers blueprint construction, routing, branching, merging, filtering, iterations, aggregations, error handling, scheduling, webhooks, data stores, subscenarios, bundles, AI agents, and provider disambiguation.

References: 18 files (see repository structure above)

## Key MCP Tools

### Remote Make server (`make`)

**Discovery:**
- `apps_recommend` — Find relevant Make apps for a use case (one app per call)
- `app_modules_list` — List modules for an app (triggers, actions, searches)
- `app_documentation_get` — Get detailed app documentation

**Module configuration:**
- `app-module_get` — Get module interface/schema (use `outputFormat: "instructions"`)
- `rpc_execute` — Resolve dynamic field options (dropdowns, resource lists)
- `validate_module_configuration` — Validate module config before committing

**Connections & keys:**
- `connections_list` — List existing connections (filter by `accountName`, not app name)
- `credential_requests_create` — Start OAuth flow for new connection
- `credential_requests_get` — Poll for credential request completion
- `keys_list` — List API keys

**Components:**
- `hooks_create` / `hooks_list` — Create and list webhooks
- `data-structures_create` / `data-structures_list` — Create and list data structures
- `data-stores_create` / `data-stores_list` — Create and list data stores

**Lifecycle:**
- `scenarios_create` — Create a scenario from a blueprint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [integromat/make-skills](https://github.com/integromat/make-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
