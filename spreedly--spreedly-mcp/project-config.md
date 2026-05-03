---
trigger: always_on
description: This document is for AI coding agents that work on the Spreedly MCP **codebase itself**. If you are an agent _using_ the MCP tools to process payments, your guidance comes from the server's `instructions` field delivered during the MCP initialize handshake (source: `src/instructions.ts`).
---

# Spreedly MCP -- Developer Agent Reference

This document is for AI coding agents that work on the Spreedly MCP **codebase itself**. If you are an agent _using_ the MCP tools to process payments, your guidance comes from the server's `instructions` field delivered during the MCP initialize handshake (source: `src/instructions.ts`).

## Project Overview

This is an MCP (Model Context Protocol) server that exposes the Spreedly payments API as tools for AI agents. It runs over stdio and is distributed as an npm package.

## Architecture

### Source Layout

```
src/
├── bin.ts                 # CLI entry point (reads env vars, starts stdio server)
├── server.ts              # MCP server creation, tool registration, passes instructions
├── instructions.ts        # SERVER_INSTRUCTIONS constant -- agent-facing guidance delivered via MCP
├── domains/               # One folder per Spreedly API domain
│   ├── index.ts           # Aggregates allTools from every domain
│   ├── gateways/          # tools.ts, schemas.ts
│   ├── transactions/
│   ├── paymentMethods/
│   ├── certificates/
│   ├── environments/
│   ├── merchantProfiles/
│   ├── subMerchants/
│   ├── events/
│   ├── protection/
│   ├── sca/
│   ├── cardRefresher/
│   └── networkTokenization/
├── security/
│   ├── descriptions.ts    # Per-tool description strings (behavioral guidance lives here)
│   ├── toolPolicy.ts      # Tool categories, access control, CATEGORY_GUIDANCE tags
│   ├── middleware.ts       # wrapHandler -- validation, error formatting, audit logging
│   ├── sanitizer.ts       # Input sanitization
│   └── audit-logger.ts    # Audit logging
├── transport/             # HTTP transport abstraction for the Spreedly API
│   ├── SpreedlyHttpTransport.ts
│   ├── types.ts
│   └── errors.ts
└── types/
    └── shared.ts          # ToolDefinition interface (name, description, schema, annotations, handler, parseError)
```

### Key Patterns

**ToolDefinition** (`src/types/shared.ts`): Every tool is a plain object with `name`, `description`, `schema` (Zod), `annotations` (MCP ToolAnnotations), a `handler` function, and an optional `parseError` hook. Tools are collected in `src/domains/index.ts` as `allTools`. The `parseError` hook lets a tool override how `SpreedlyError` responses are extracted into structured agent-facing output; when omitted, the default parser extracts `fieldErrors`, `transactionToken`, and `retryAfterMs` from typed error subclass properties.

**Tool annotations**: Each tool declares MCP-native `annotations` (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`) directly on its definition. These are per-tool, not per-category. The server passes them to `registerTool`. Each annotation has a comment explaining why it was set. When adding or modifying annotations, apply these principles:

- **`readOnlyHint`**: `true` for any GET endpoint. When set, `destructiveHint` and `idempotentHint` are irrelevant per the MCP spec, so omit them.
- **`destructiveHint`**: `true` if the tool can overwrite or delete existing data. All PUT/PATCH/DELETE endpoints are destructive because they replace existing field values. POST endpoints that only create new resources are _not_ destructive (additive). Financial operations (authorize, capture, void, credit) are destructive because they irreversibly alter transaction state.
- **`idempotentHint`**: `true` only if repeating the call with the same arguments has _no additional effect on Spreedly's state_. PUT/PATCH updates are idempotent (same payload converges to same state). Retain operations are idempotent (flag is already set). POST endpoints that create new records or new transaction objects are _not_ idempotent — this includes `verify` and `store`, which each create a new transaction record even though their external side-effects may seem repeatable.
- **`openWorldHint`**: `true` only if the tool reaches external systems _beyond Spreedly_ (payment processors, card networks, SCA/protection providers). CRUD operations on Spreedly-managed resources (gateways, payment methods, certificates, environments, etc.) are closed-world even though they call the Spreedly API — the domain of interaction is bounded. Vaulting a payment method (`_create`) and recaching CVV (`_recache`) are closed-world because they operate on Spreedly's vault, not an external gateway.

**Descriptions** (`src/security/descriptions.ts`): All behavioral guidance for consuming agents lives in per-tool descriptions. Financial tools include immutability language ("Operator-provided values are immutable. Never retry with altered financial parameters."). This is the primary mechanism for steering agent behavior.

**Categories are purely access control** (`src/security/toolPolicy.ts`): `TOOL_CATEGORIES` maps tool names to one of four categories. `CATEGORY_GUIDANCE` only contains toggle tags like `[Enabled by TRANSACTION_INITIATION_ENABLED=true]` -- it does NOT contain behavioral guidance. Behavioral guidance belongs in per-tool descriptions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spreedly/spreedly-mcp](https://github.com/spreedly/spreedly-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
