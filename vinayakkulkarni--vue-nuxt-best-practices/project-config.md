---
trigger: always_on
description: > This file is auto-generated. Do not edit directly.
---

# Nuxt Agent-Ready Best Practices - Complete Reference

> This file is auto-generated. Do not edit directly.
> Edit individual rule files in the `rules/` directory and run `pnpm build`.

# Nuxt Agent-Ready Best Practices

Guidelines for making a Nuxt 4 site **operable by autonomous AI agents** — measured by the [isitagentready.com](https://isitagentready.com) scanner (Cloudflare's "Is Your Site Agent-Ready?"). This is a different axis from GEO: GEO is about being _cited_ in AI answers; agent-readiness is about being _operated_ — an agent authenticating, discovering your API, calling your tools, and taking action.

Proven on production Nuxt 4 + Nitro `cloudflare_module` Workers:

- A **marketing site** (only public POST endpoints, no auth/MCP server): **21 → 50+ (Level 1 → Level 4 "Agent-Integrated")** — the auth + MCP surfaces are honesty-gated OFF (see below).
- A **full platform** with a real OAuth server (Better-Auth oauth-provider) + a real remote MCP server: **21 → 100/100 (Level 5 "Agent-Native"), all 14 checks green**. The auth + MCP surfaces are legitimately publishable there, which is what unlocks the last ~40 points.

**The ceiling is set by what you actually run, not by effort.** A marketing site tops out around Level 4 and that is the _correct_ score — do not fabricate an auth server to chase 100 (see THE HONESTY RULE). Only a site with a real authorization server and a real MCP server can honestly reach Level 5.

## GEO vs Agent-Readiness (know the difference)

|          | GEO (`nuxt-geo-best-practices`)                            | Agent-Readiness (this skill)                                           |
| -------- | ---------------------------------------------------------- | ---------------------------------------------------------------------- |
| Goal     | Be **cited** in AI answers                                 | Be **operated** by agents                                              |
| Question | "Will ChatGPT mention me?"                                 | "Can an agent call my tools and act?"                                  |
| Levers   | llms.txt, crawler allowlist, RAG content, JSON-LD entities | MCP/WebMCP, API/skill discovery, agent auth, DNS-AID, agentic commerce |

**Shared primitives live in the GEO skill.** `robots.txt` AI-crawler allowlisting, `llms.txt`/`llms-full.txt`, and the XML sitemap are covered by `nuxt-geo-best-practices` (rules `ai-robots-allowlist`, `ai-llms-txt`, `ai-sitemap`). The isitagentready scanner scores those too — set them up via the GEO skill first, then apply this skill for the agent-operation layer on top.

## When to Apply

- Raising a site's score on `https://isitagentready.com/<domain>`
- Supporting `Accept: text/markdown` content negotiation for agents
- Advertising resources via RFC 8288 `Link` headers
- Publishing an RFC 9727 API catalog (`/.well-known/api-catalog`)
- Publishing an Agent Skills discovery index (`/.well-known/agent-skills/index.json`)
- Exposing site actions to in-browser agents via WebMCP (`navigator.modelContext`)
- Publishing an MCP Server Card (only if you run a real MCP server)
- Publishing OAuth/OIDC discovery for agent auth (only if you run a real auth server)
- Publishing DNS-AID records (`_index._agents.<domain>`) + DNSSEC

## THE HONESTY RULE (load-bearing — read before publishing anything)

**Only publish discovery for services that actually exist.** A discovery document that sends an agent to a dead end — a `/.well-known/openid-configuration` with no auth server behind it, an MCP Server Card whose transport endpoint 404s, a `_mcp._agents` DNS record with no MCP server — is **worse than a lower score**. Agents will try to use it and fail.

This mirrors the "never fake customers/logos/scale" rule: a fabricated capability that fails on first contact destroys trust. On a marketing site with only public POST endpoints, **skip** OAuth/OIDC discovery, oauth-protected-resource, auth.md, and the MCP Server Card — they belong on the app/console domain (real auth server) or require building a real MCP server. Decline these explicitly and say why.

**The flip side — when you DO run the real thing, publish it fully.** If your site runs a real OAuth authorization server (e.g. Better-Auth `oauth-provider` plugin) and a real remote MCP server, the auth + MCP surfaces are no longer dishonest — they are the highest-value checks and unlock Level 5. maps.guru scored 100/100 precisely because those services exist. The honesty rule cuts both ways: don't fake it, but don't under-claim a real capability either.

## THE SCANNER-BEHAVIOR RULES (what actually flips a check green)

Passing the harder checks is NOT "publish the file and move on" — the isitagentready scanner inspects **content and runtime behavior**, not just presence. Three non-obvious behaviors cost real time to discover:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinayakkulkarni/vue-nuxt-best-practices](https://github.com/vinayakkulkarni/vue-nuxt-best-practices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
