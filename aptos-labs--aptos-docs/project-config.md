---
trigger: always_on
description: This document provides essential guidelines for AI agents working on the Aptos Developer Documentation repository.
---

# CLAUDE.md - AI Agent Guidelines for Aptos Documentation

This document provides essential guidelines for AI agents working on the Aptos Developer Documentation repository.

## Project Overview

This repository contains the official Aptos Developer Documentation, built using [Astro](https://astro.build/) and [Starlight](https://starlight.astro.build/). Published languages include English and Chinese (zh). Agent workflows here do **not** include creating or updating Spanish (`es`) documentation.

## Machine-readable documentation for agents

Production docs are indexed for LLMs and coding agents at [https://aptos.dev/llms.txt](https://aptos.dev/llms.txt) (same content as [https://aptos.dev/.well-known/llms.txt](https://aptos.dev/.well-known/llms.txt)). For IDE access to Aptos APIs and on-chain data, use the Aptos MCP server (`npx @aptos-labs/aptos-mcp`); see the live [AI tools](https://aptos.dev/build/ai) page.

## Agent discovery & readiness (keep fresh!)

The site advertises a full set of agent-discovery endpoints. Treat these as a single surface — if you touch one, audit the rest, update the matching docs, regenerate any required artifacts (`pnpm build:middleware-matcher` for matcher changes; `pnpm build:middleware` when the middleware bundle needs to ship), and then re-run `pnpm test tests/agent-discovery.test.ts tests/markdown-negotiation.test.ts`. The guardrail tests regenerate the matcher on demand, so a clean checkout still passes.

| Concern                              | Source of truth                                        | Spec / reference                                                                 |
| ------------------------------------ | ------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Global `Link` response header        | `vercel.json` → `headers[source="/(.*)"]`              | [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288)                               |
| In-page discovery `<link rel="…">`   | `src/starlight-overrides/Head.astro`                   | RFC 8288                                                                         |
| API catalog                          | `public/.well-known/api-catalog`                       | [RFC 9727](https://www.rfc-editor.org/rfc/rfc9727)                               |
| MCP Server Card                      | `public/.well-known/mcp/server-card.json`              | [SEP-2127](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2127) |
| Agent Skills index                   | `public/.well-known/agent-skills/index.json`           | [Agent Skills Discovery RFC v0.2.0](https://github.com/cloudflare/agent-skills-discovery-rfc) |
| Agent Skills regeneration            | `scripts/generate-agent-skills-index.mjs`              | Pins `sha256:` digests from `aptos-labs/aptos-agent-skills@main`                 |
| Content Signals                      | `public/robots.txt` → `Content-Signal:` line           | [contentsignals.org](https://contentsignals.org/)                                |
| OAuth Protected Resource Metadata    | `public/.well-known/oauth-protected-resource` (only the [Aptos Testnet Faucet](https://aptos.dev/network/faucet) is OAuth-protected) | [RFC 9728](https://www.rfc-editor.org/rfc/rfc9728)                               |
| OIDC / OAuth 2.0 discovery           | `public/.well-known/openid-configuration`, `public/.well-known/oauth-authorization-server` | [OpenID Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html), [RFC 8414](https://www.rfc-editor.org/rfc/rfc8414) |
| Markdown content negotiation         | `src/middlewares/markdown-negotiation.ts` + `src/vercel-middleware.ts` | Rewrites `Accept: text/markdown` requests to the `.md` export.      |
| Middleware matcher + bundle          | `scripts/generate-middleware-matcher.js`, `scripts/generate-middleware.js`, committed `middleware.js` | Matcher covers the locales + route prefixes middleware should run on. `generate-middleware.js` also copies the bundled middleware to the repo root so `scripts/generate-middleware-function.js` ships it — never hand-edit `middleware.js`. |
| MCP card regeneration                | Hand-maintained `public/.well-known/mcp/server-card.json` pinned to a real `@aptos-labs/aptos-mcp` version | Bump the `version` field and both `packages[0].version` + `packages[0].transport.args` whenever the npm package is released; never use `latest`. |
| WebMCP tools                         | `src/scripts/webmcp-register.ts` (+ `src/types/webmcp.d.ts`, `src/components/WebMcpRegistration.astro`) | [WebMCP draft](https://webmachinelearning.github.io/webmcp/)        |
| User-facing explainer                | `src/content/docs/build/ai.mdx` and `src/content/docs/zh/build/ai.mdx` | Keep the endpoint table and example `curl` in sync.                 |

Keep-fresh checklist when editing any of the above:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aptos-labs/aptos-docs](https://github.com/aptos-labs/aptos-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
