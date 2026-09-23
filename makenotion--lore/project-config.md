---
trigger: always_on
description: > Read the root `AGENTS.md` first. This file covers the Notion SDK integration
---

# AGENTS.md -- src/notion/

> Read the root `AGENTS.md` first. This file covers the Notion SDK integration
> layer only.

## Purpose

This directory contains the code that directly touches the Notion API: client
configuration, database schemas, property extractors, vault setup, relation
hydration, and RunTool adapters. Domain logic belongs in `src/core/`.

## Documentation Map

| Need                                                          | Read                                                                                                   |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| SDK v5 request and response shapes                            | [`docs/notion-sdk-v5.md`](../../docs/notion-sdk-v5.md)                                                 |
| Rate-limit gates, endpoint overrides, and call-site checklist | [`docs/notion-rate-limit.md`](../../docs/notion-rate-limit.md)                                         |
| RunTool routing and rollback entry point                      | [`runtool/README.md`](runtool/README.md)                                                               |
| RunTool API contract                                          | [`runtool/contract.md`](runtool/contract.md)                                                           |
| RunTool consumer behavior and fallbacks                       | [`runtool/consumers.md`](runtool/consumers.md)                                                         |
| RunTool historical evidence and phase logs                    | [`docs/archive/runtool-evidence.md`](../../docs/archive/runtool-evidence.md)                           |
| Auth token resolution before this layer receives a bearer     | [`src/auth/AGENTS.md`](../auth/AGENTS.md) and [`docs/authentication.md`](../../docs/authentication.md) |
| Service behavior built on these primitives                    | [`src/core/AGENTS.md`](../core/AGENTS.md)                                                              |

## Files

| File                     | Responsibility                                                                          |
| ------------------------ | --------------------------------------------------------------------------------------- |
| `client.ts`              | Creates a configured `Client` instance with custom timeout and User-Agent.              |
| `page-id-schema.ts`      | Shared Zod schema for Notion page IDs, including undashed URL-form normalization.       |
| `rate-limit.ts`          | Wraps the Notion client in request-rate, concurrency, and shared-backoff gates.         |
| `schema.ts`              | Defines database property configs, property name constants, and page property builders. |
| `extractors.ts`          | Provides typed property value extractors for `PageObjectResponse`.                      |
| `query-response.ts`      | Guards data-source query response shape and preserves validation payload errors.        |
| `relation-properties.ts` | Paginates relation property values when page responses are truncated.                   |
| `setup.ts`               | Creates and verifies the five-database vault structure.                                 |
| `runtool/`               | Hosts the quarantined RunTool integration, public wrappers, feature flags, and tests.   |

## RunTool quarantine

`runtool/` is the home for Lore's quarantined integration with Notion's
internal `POST /v1/tools/run` API. Keep this guide to routing pointers:

- `runtool/README.md` owns the current default, operator rollback path, and
  doc index.
- `runtool/contract.md` owns endpoint, envelope, auth/capability, rate-limit,
  response-shape, pin, and error-vocabulary contracts.
- `runtool/consumers.md` owns per-consumer surfaces and fallback behavior for
  `create_pages`, `update_page`, `query_data_sources`, and `search`.
- `docs/archive/runtool-evidence.md` preserves old phase logs, verification
  runs, and default-on evidence.

The canonical name for the 403 capability-rejection kind is
`restricted_resource`, matching `APIErrorCode.RestrictedResource`. RunTool
consumers must use the same spelling.

## Property Extractors Pattern

`extractors.ts` provides typed helper functions for pulling values out of
Notion page properties. Every core service uses these instead of inlining
property access logic.

| Extractor                       | Input property type | Returns          |
| ------------------------------- | ------------------- | ---------------- |
| `extractTitle(prop)`            | `title`             | `string`         |
| `extractRichText(prop)`         | `rich_text`         | `string`         |
| `extractSelect(prop, fallback)` | `select`            | `string`         |
| `extractMultiSelect(prop)`      | `multi_select`      | `string[]`       |
| `extractRelationIds(prop)`      | `relation`          | `string[]`       |
| `extractDate(prop)`             | `date`              | `string \| null` |

`extractRelationIds()` is intentionally synchronous and only reads IDs already
present on a page response. When a relation property has `has_more: true`, use

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makenotion/lore](https://github.com/makenotion/lore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
