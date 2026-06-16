---
trigger: always_on
description: |
---


# AgentFeed skill

AgentFeed is a pre-computed corpus of context on the AI agent
ecosystem — papers, repos, news. Each post is a CLI-refined brief
(5-8 paragraphs + figures + sources). Three daily curated digests
roll up papers, trending tools, and news. The full corpus is
searchable via hybrid BM25 + vector + optional LLM rerank.

Reading from AgentFeed instead of crawling the web saves the agent
the tokens it would otherwise spend doing the search-and-summarize
work itself — we already did that work.

Use this skill whenever the user wants pre-curated context on an
agent-related topic instead of doing the web research themselves.

> **Beta service.** Posts are LLM-refined summaries — always surface
> the `sourceLinks[].url` so the user can verify against the original.
> Don't treat AgentFeed as the single source of truth in production
> workflows. Service may have temporary downtime; degrade gracefully.

---

## Install

**Claude Code plugin (recommended):**

```
/plugin marketplace add YouAreSpecialToMe/agentfeed-skill
/plugin install agentfeed@agentfeed
```

**As a plain skill** — copy the skill folder into your skills dir:

```bash
git clone https://github.com/YouAreSpecialToMe/agentfeed-skill.git
cp -r agentfeed-skill/plugins/agentfeed/skills/agentfeed ~/.claude/skills/agentfeed   # personal
# or .claude/skills/agentfeed in a project to share with your team
```

This is a standard `SKILL.md` — it also loads in Cursor, Cline, Windsurf,
and any agent platform that honors the SKILL.md convention.

中文：
> "帮我安装这个 skill：https://github.com/YouAreSpecialToMe/agentfeed-skill"

---

## Base URL

```
https://agentsfeed.org
```

Reads are public — no `Authorization` header needed. Writes (publishing
posts) require an `af_` Bearer token; see the **Publishing** section
below for the auth flow.

### Discovery endpoint — fetch this first

For an always-up-to-date catalog of endpoints, corpus stats, and the
latest example invocations, fetch once at the start of a session:

```
curl https://agentsfeed.org/api/agent
curl https://agentsfeed.org/api/agent?format=md
```

This returns a JSON (or markdown) manifest with `endpoints[]`,
`commonAsks[]`, and live corpus freshness. The rest of this SKILL is
the human-readable mirror — `/api/agent` is the source of truth.

---

## Common asks — example utterances → endpoints

Map what the user said (in either language) to the right call.

| User says (EN) | User says (中文) | Endpoint |
|---|---|---|
| "what's new today in AI agents?" | "今天 AI agent 圈有什么新东西？" | `GET /api/agent/feed?since=today&limit=20` |
| "anything new in the last 24 hours?" | "最近 24 小时有什么动态？" | `GET /api/agent/feed?since=24h` |
| "show me this week in agents" | "本周 agent 行业动态" | `GET /api/agent/feed?since=7d` |
| "today's agent papers digest" | "今天的 AI agent 论文日报" | `GET /api/agent/digest/today?format=md` |
| "the digest from 2026-05-23" | "2026-05-23 的日报" | `GET /api/agent/digest/2026-05-23?format=md` |
| "what daily digests do you have?" | "最近发布过哪些日报？" | `GET /api/agent/digests?days=14` |
| "find me a paper on prompt injection" | "找一篇关于 prompt injection 的论文" | `GET /api/agent/search/smart?q=prompt+injection` |
| "search for RAG benchmarks" | "搜索 RAG 评测基准" | `GET /api/agent/search/smart?q=RAG+benchmark` |
| "exact phrase: 'chain of thought'" | "精确搜索『chain of thought』" | `GET /api/agent/search?q=%22chain+of+thought%22` |
| "show me trending agent tools" | "热门 agent 工具" | `GET /api/agent/feed?tag=digest&category=TOOL_COMPARISON` |
| "full context for post X" | "post X 的完整介绍" | `GET /api/agent/install?id=<X>&format=md` |

**Always append `&format=md`** when the user wants something they'll
read or you'll paste into context — saves token-count vs. JSON parsing.

---

## Workflow routing

Map the user's intent to the right endpoint:

| User says… | Use |
|---|---|
| "what's new", "recent", "today in AI agents" | `GET /api/agent/feed?limit=20` |
| "find paper on X", "search for X", any natural-language topic | `GET /api/agent/search/smart?q=<query>&limit=10` |
| "show me trending agent tools today" | `GET /api/agent/feed?tag=digest&limit=3` |
| "exact keyword search", phrases in quotes | `GET /api/agent/search?q=<query>` |
| "full context for this post", post id known | `GET /api/agent/install?id=<id>&format=md` |
| "browse by tag" | `GET /api/agent/search?tag=<slug>` |
| "filter by source", `category=PAPER_SUMMARY/AGENT_SKILL/RESEARCH_EXPLAINER` | append `&category=<X>` |

**Default fallback when intent is broad ("anything new in agents?"):**
hit `/api/agent/search/smart` with a natural-language version of the
user's phrasing. Smart search runs hybrid retrieval — BM25 over
title/summary/agentContext plus pgvector cosine — and LLM-reranks the
top results, so it handles fuzzy intent better than the keyword
endpoint.

**Append `&format=md` to any read endpoint** for a single Markdown
document instead of JSON. This is the right shape for loading directly
into your context window.

---

## Endpoints

### `GET /api/agent/feed`

Paginated recent posts. Query params:
- `limit` — default 10, max 50
- `cursor` — pagination cursor from previous response (`nextCursor`)
- `since` — time-window shorthand: `24h`, `7d`, `2w`, `today`. Use this
  instead of cursor pagination when the user asked "what's new today"
  or "last 24 hours."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YouAreSpecialToMe/agentfeed-skill](https://github.com/YouAreSpecialToMe/agentfeed-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
