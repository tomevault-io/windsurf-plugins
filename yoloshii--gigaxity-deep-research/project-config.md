---
trigger: always_on
description: This is the agent reference for Gigaxity Deep Research, an open-source deep research MCP server for Claude Code, Hermes, Cursor, and other MCP-compatible agents. Qwen3-30B-A3B-Thinking runs via OpenRouter, the Triple Stack search MCPs (Context7, Exa, Jina) handle web/docs/code retrieval, and the bundled `research-workflow` skill routes queries to the right tool per query class.
---

# Gigaxity Deep Research — Agent Reference

This is the agent reference for Gigaxity Deep Research, an open-source deep research MCP server for Claude Code, Hermes, Cursor, and other MCP-compatible agents. Qwen3-30B-A3B-Thinking runs via OpenRouter, the Triple Stack search MCPs (Context7, Exa, Jina) handle web/docs/code retrieval, and the bundled `research-workflow` skill routes queries to the right tool per query class.

This file is loaded by Claude Code (`CLAUDE.md`) and other MCP-compatible agents (`AGENTS.md` is byte-identical). It documents how to operate the six MCP tools this server exposes (two primitives plus four deep-research tools) and how to plug them into the broader deep research stack.

If your harness loads a global `CLAUDE.md` or `AGENTS.md` (Claude Code, Codex, Cursor, Hermes, etc.), copy the **instruction block** at the bottom of this file into that global file. For standalone agents that take a system prompt instead, paste the block directly into the system prompt. That single block makes any compatible agent automatically route research queries through this MCP plus the six companion MCPs (Context7, Exa, Exa Answer, Jina, Brightdata fallback, gptr-mcp) in the full deep research stack.

---

## Tool surface

The MCP server exposes **two primitives** plus **four deep-research tools** — six tools total. Pick a primitive when you want raw or combined behavior in one call; pick a deep-research tool when you want to drive discovery, synthesis, or reasoning as a discrete step.

**Primitives**

| Tool | Use for | Token cost (typical) |
|---|---|---|
| `mcp__gigaxity-deep-research__search` | Raw multi-source aggregation (SearXNG + Tavily + LinkUp + RRF). No LLM call. | 0 LLM tokens; search-API quotas only |
| `mcp__gigaxity-deep-research__research` | Combined search + synthesis with citations in a single call. The simple pipeline. | ~3000–8000 |

**Deep-research tools**

| Tool | Use for | Token cost (typical) |
|---|---|---|
| `mcp__gigaxity-deep-research__ask` | Quick conversational answer; speed > depth (direct LLM, no search hop) | ~500–1500 |
| `mcp__gigaxity-deep-research__discover` | Cold-start exploration; surfaces explicit/implicit/related/contrasting angles + gap detection | ~2000–5000 |
| `mcp__gigaxity-deep-research__synthesize` | Citation-aware fusion of pre-gathered content; CRAG quality gate, contradiction surfacing | ~5000–10000 |
| `mcp__gigaxity-deep-research__reason` | Deep synthesis with explicit chain-of-thought depth control over pre-gathered content | ~5000–15000 |

All six tools accept an optional `openrouter_api_key` parameter for per-request key override (multi-tenant deployments). REST callers can use the `X-OpenRouter-Api-Key` header for the same purpose.

---

## When to call which tool

```
Query class?
├── "what is X right now / latest version" (single fact, speed-critical)
│     → ask
│
├── "tell me about X" (cold start, no prior context, want breadth)
│     → discover
│
├── "compare X vs Y" or "best practice for X" (cross-source synthesis, citations matter)
│     → synthesize
│
└── "why did X happen" or "explain the reasoning behind X" (CoT reasoning matters)
      → reason
```

For the full classification tree across the **entire** Triple Stack (when to use Context7, Exa, Jina, Brightdata fallback alongside this MCP), see the bundled [`skills/research-workflow/SKILL.md`](skills/research-workflow/SKILL.md).

---

## Environment variables

All variables are prefixed `RESEARCH_`. Set in `.env` (gitignored) or pass via the MCP `env` config block.

| Variable | Default | Purpose |
|---|---|---|
| `RESEARCH_LLM_API_BASE` | `https://openrouter.ai/api/v1` | LLM endpoint. For local inference, set to `http://localhost:8000/v1` etc. |
| `RESEARCH_LLM_API_KEY` | *(empty — required)* | OpenRouter or local-server API key |
| `RESEARCH_LLM_MODEL` | `qwen/qwen3-30b-a3b-thinking-2507` | Any OpenAI-compatible chat-completions model |
| `RESEARCH_LLM_TEMPERATURE` | `0.85` | |
| `RESEARCH_LLM_TOP_P` | `0.95` | |
| `RESEARCH_LLM_MAX_TOKENS` | `16384` | |
| `RESEARCH_LLM_TIMEOUT` | `120` | Seconds |
| `RESEARCH_SEARXNG_HOST` | `http://localhost:8888` | Primary search source — required |
| `RESEARCH_SEARXNG_ENGINES` | `brave,bing,duckduckgo,startpage,mojeek,wikipedia` | Matches the bundled SearXNG `settings.yml.example` enabled list |
| `RESEARCH_TAVILY_API_KEY` | *(empty)* | Optional additional connector — runs in parallel with SearXNG, RRF-fused |
| `RESEARCH_LINKUP_API_KEY` | *(empty)* | Optional additional connector — runs in parallel with SearXNG, RRF-fused |
| `RESEARCH_DEFAULT_TOP_K` | `10` | Results per source |
| `RESEARCH_RRF_K` | `60` | RRF fusion constant |
| `RESEARCH_HOST` | `127.0.0.1` | REST mode only. Default loopback; bind `0.0.0.0` only behind an authenticated reverse proxy. |
| `RESEARCH_PORT` | `8000` | REST mode only |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoloshii/gigaxity-deep-research](https://github.com/yoloshii/gigaxity-deep-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
