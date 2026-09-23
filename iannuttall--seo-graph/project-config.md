---
trigger: always_on
description: > This file is for AI coding agents (Claude Code, Cursor, Copilot, etc.)
---

# AGENTS.md — seo-graph

> This file is for AI coding agents (Claude Code, Cursor, Copilot, etc.)
> building sites that use `@iannuttall/seo-graph-core` and/or
> `@iannuttall/seo-graph-astro`. It explains what the library does, how the
> pieces fit together, and which schema.org entities to use for every
> common site type.

## What this library does

seo-graph makes a site legible to AI agents and search engines from one
toolkit, in two layers:

1. **Schema graphs.** Valid, linked schema.org JSON-LD `@graph` arrays from
   typed inputs. Instead of hand-writing JSON-LD (error-prone) or copying
   snippets from schema.org docs (inconsistent), you call piece builders that
   return strongly-typed entities, then wrap them in a `@graph` envelope.
2. **Agent markdown.** Deterministic Markdown representations of every page —
   rendered from the built HTML for arbitrary pages (with a strict exclusion
   contract so decorative markup never leaks) and straight from the source
   entry for content-collection pages — plus `llms.txt`, a route manifest,
   IndexNow, and an HTTP content-negotiation handler so agents can request
   `Accept: text/markdown` at the canonical URL.

Two packages:

- **`@iannuttall/seo-graph-core`** — Pure TypeScript, no framework
  dependency. Piece builders, ID factory, graph assembler, deduplication,
  the built-HTML → Markdown renderer, collection markdown renderer, route
  mapping, manifests, `llms.txt`, git-based lastmod, IndexNow manifest
  hashing. Use this from any runtime.
- **`@iannuttall/seo-graph-astro`** — Astro layer. `agentMarkdown()` build
  integration, collection markdown endpoints, schema endpoints and schema
  map, IndexNow key route, RFC 9727 api-catalog, Zod content helpers, and a
  Cloudflare Worker content-negotiation handler (`./cloudflare` subpath).

Portions of the schema layer derive from
[jdevalk/seo-graph](https://github.com/jdevalk/seo-graph) (MIT, see NOTICE).
The agent-markdown pipeline is original work first built for
[seoskill.dev](https://seoskill.dev).

---

## Contents

**Schema core** — concepts and builders for the JSON-LD graph:

- [Architecture](#architecture)
- [Installation](#installation)
- [The @id system](#the-id-system)
- [Piece builders reference](#piece-builders-reference)

**Recipes and patterns** — how to model real sites:

- [Site type recipes](#site-type-recipes)
- [Trust and credibility signals](#trust-and-credibility-signals)
- [Choosing the right Article subtype](#choosing-the-right-article-subtype)
- [Actions: telling agents what they can do](#actions-telling-agents-what-they-can-do)
- [Multi-type entities](#multi-type-entities)
- [Rich Organization patterns](#rich-organization-patterns)
- [Rich Person patterns](#rich-person-patterns)

**Agent markdown** — deterministic Markdown for every page:

- [The agent markdown pipeline](#the-agent-markdown-pipeline)
- [The content selection contract](#the-content-selection-contract)
- [Astro build integration](#astro-build-integration)
- [Live markdown for server-rendered pages](#live-markdown-for-server-rendered-pages)
- [Collection markdown endpoints](#collection-markdown-endpoints)
- [Cloudflare content negotiation](#cloudflare-content-negotiation)

**Discovery surfaces** — endpoints agents can find and consume:

- [Schema endpoints and the schema map](#schema-endpoints-and-the-schema-map)
- [llms.txt and the route manifest](#llmstxt-and-the-route-manifest)
- [IndexNow](#indexnow)

**Reference:**

- [Common mistakes](#common-mistakes)
- [Validating your output](#validating-your-output)
- [Repository structure](#repository-structure)

---

## Architecture

```txt
┌────────────────────────────────────────────────────────────┐
│              @iannuttall/seo-graph-astro                   │
│  agentMarkdown()   createMarkdownEndpoint()                │
│  createSchemaEndpoint()  createSchemaMap()                 │
│  createIndexNowKeyRoute()  createApiCatalog()              │
│  seoSchema()/imageSchema()   ./cloudflare handler          │
└──────────────────────────┬─────────────────────────────────┘
                           │ builds on
┌──────────────────────────▼─────────────────────────────────┐
│              @iannuttall/seo-graph-core                    │
│                                                            │
│  Schema graph            Agent markdown                    │
│  ─────────────           ────────────────                  │
│  piece builders          renderAgentMarkdown (built HTML)  │
│  makeIds                 renderMarkdownAlternate (source)  │
│  assembleGraph           routes / manifests / llms.txt     │
│  deduplicateByGraphId    gitLastmod / IndexNow hashing     │
└────────────────────────────────────────────────────────────┘
```

Core is pure and framework-agnostic: a future `seo-graph-nextjs` (or
Eleventy, SvelteKit, Hono…) layer would sit beside the Astro package and
reuse everything in core. Anything that is pure TypeScript belongs in core;
only genuinely framework-bound code (Astro `APIRoute` factories, the build
hook) lives in the framework layer.

Markdown has two sources feeding one output contract:

- **Built-HTML path** — `agentMarkdown()` walks the final build output,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iannuttall/seo-graph](https://github.com/iannuttall/seo-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
