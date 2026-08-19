---
trigger: always_on
description: > Read this file first, then read the guide for the area you are working in.
---

# Agent Reference

> Read this file first, then read the guide for the area you are working in.
> When a subsystem guide conflicts with this file, the subsystem guide wins for
> that subsystem.

## Subsystem Guides

| Area             | Guide                                          | Scope                                                                       |
| ---------------- | ---------------------------------------------- | --------------------------------------------------------------------------- |
| MCP server       | [`src/mcp/AGENTS.md`](src/mcp/AGENTS.md)       | Tool registration, error handling, server startup                           |
| Domain services  | [`src/core/AGENTS.md`](src/core/AGENTS.md)     | Service pattern, context resolution, fact invalidation                      |
| Notion SDK layer | [`src/notion/AGENTS.md`](src/notion/AGENTS.md) | Client, schema, extractors, vault setup, Notion doc routing                 |
| CLI              | [`src/cli/AGENTS.md`](src/cli/AGENTS.md)       | CLI routing, command inventory, detailed guide pointers                     |
| Hook runner      | [`src/hooks/AGENTS.md`](src/hooks/AGENTS.md)   | Hook routing, handler registration, doc precedence                          |
| Auth layer       | [`src/auth/AGENTS.md`](src/auth/AGENTS.md)     | ntn / PAT auth, `auth.json` contract, vault preflight, token classification |

## Detailed Guides

| Guide                                                                                                                                                                                        | Use it for                                                                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| [`docs/development.md`](docs/development.md)                                                                                                                                                 | Architecture, commands, conventions, stability rules, troubleshooting                                                        |
| [`docs/authentication.md`](docs/authentication.md)                                                                                                                                           | Auth priority chain, ntn behavior, `auth.json` contract, rate limits, troubleshooting                                        |
| [`docs/profiles.md`](docs/profiles.md)                                                                                                                                                       | Default profile selector, profile-owned taxonomy/schema/prompts, no-singleton threading                                      |
| [`docs/memory-workflows.md`](docs/memory-workflows.md)                                                                                                                                       | Lore memory/fact/decision/task workflow, topic keys, digest                                                                 |
| [`docs/conflict-detection.md`](docs/conflict-detection.md)                                                                                                                                   | `lore conflicts scan` workflow and compare-verdict contract                                                                  |
| [`docs/memory-debt.md`](docs/memory-debt.md)                                                                                                                                                 | `lore debt scan` / `create-tasks` audit categories, scoring, recommended maintenance cadence, idempotency contract           |
| [`docs/topology.md`](docs/topology.md)                                                                                                                                                       | `lore status` topology section, health states, recovery workflow                                                             |
| [`docs/topology-inheritance.md`](docs/topology-inheritance.md)                                                                                                                               | Read-upstream wake-up rendering, trust containment, design rules, opt-out behavior, promotion-target read-upstream carve-out |
| [`docs/topology-promotion.md`](docs/topology-promotion.md)                                                                                                                                   | Promotion workflow, audit block, guards, dry run, source validation, idempotency, promoter identity, implementation pointers |
| [`docs/notion-sdk-v5.md`](docs/notion-sdk-v5.md), [`docs/notion-rate-limit.md`](docs/notion-rate-limit.md)                                                                                   | Notion SDK v5 shape differences, rate-limit gates, endpoint overrides, call-site checklist                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makenotion/lore](https://github.com/makenotion/lore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
