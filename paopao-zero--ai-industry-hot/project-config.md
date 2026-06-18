---
trigger: always_on
description: Fetch and analyze AI industry news from AIHOT and other cited sources, then produce Chinese briefings with AI industry-chain impact mapping, bullish/bearish/neutral/watchlist judgments, push rationale, and risk notes. Use when the user asks about today's AI hotspots, recent AI industry news, OpenAI/Anthropic/Nvidia/Agent/GPU updates, AI产业链利好利空, or AI news suitable for finance/industry research briefings.
---


# AI Industry HOT

Use this skill to turn AI industry news into a concise Chinese industry-chain briefing. Focus on information screening, factual summarization, impact-path analysis, and risk disclosure. Do not provide investment advice, buy/sell recommendations, target prices, or deterministic price predictions.

## Workflow

1. Determine the user's time window and topic.
   - Default: latest selected AIHOT items from the last 24 hours.
   - If the user asks for "最近几天", set `since` to that window, with a maximum of 7 days for the items endpoint.
   - If the user asks for a company or theme, use `q`, such as `OpenAI`, `Anthropic`, `Nvidia`, `Agent`, `GPU`, `MCP`, or `Claude`.
2. Fetch current AI news before answering. AI news is time-sensitive; do not rely only on memory.
3. Keep only items with clear industry relevance. Prioritize model releases, product launches, infrastructure, compute, cloud, agent applications, financing, M&A, regulation, open source, and technical breakthroughs.
4. Do not invent facts, links, sources, dates, companies, or market reactions. If no relevant result is returned, say so directly and suggest a narrower query.
5. Analyze each selected item through event type, industry-chain mapping, impact direction, push rationale, and risk note.
6. Write in Chinese Markdown, suitable for Feishu/WeChat/classroom presentation.

## AIHOT Data Access

Use AIHOT public REST endpoints as the primary source.

- Selected latest items:
  `GET https://aihot.virxact.com/api/public/items?mode=selected&since=<ISO_TIME>&take=50`
- All latest items, only when the user explicitly asks for 全部/完整/所有/全量:
  `GET https://aihot.virxact.com/api/public/items?mode=all&since=<ISO_TIME>&take=50`
- Keyword search:
  `GET https://aihot.virxact.com/api/public/items?q=<KEYWORD>&take=50`
- Daily briefing:
  `GET https://aihot.virxact.com/api/public/daily`
  or `GET https://aihot.virxact.com/api/public/daily/<YYYY-MM-DD>` when the user asks for a specific daily report.

When calling `/api/public/*`, include a browser-like User-Agent header to avoid access failures:

```bash
curl -H "User-Agent: Mozilla/5.0" "https://aihot.virxact.com/api/public/items?mode=selected&take=50"
```

If AIHOT is unavailable, times out, cannot be accessed, returns too little information, or returns mostly duplicate/low-quality items, do not abandon this skill or downgrade to a free-form summary. Enter fallback mode: use other public sources only when they can be cited with links, such as OpenAI Blog, Anthropic News, Google Blog, NVIDIA Blog, Microsoft Blog, company newsrooms, arXiv, Hugging Face, GitHub, Reuters, CNBC, or official regulator/government pages. Cross-verify candidate items across available sources when possible, then score and select the highest-signal items using these dimensions: source authority (official/regulator/primary sources first), cross-source confirmation, recency relative to the user's requested window, industry-chain relevance, and investment-analysis value (whether the item supports a clear 偏利好/偏利空/中性/待观察 judgment). In fallback mode, keep the same `Output Format` and item template exactly; include source links, source/time, event type, one-sentence summary, impact path, industry-chain impact, push rationale, and risk note for each selected item.

## AIHOT Routing Rules

- Route broad questions like "今天 AI 圈", "过去 24 小时大新闻", and "最近 AI 圈有啥" to the rolling items endpoint with `mode=selected` and a semantic `since` window.
- Route to the daily endpoint only when the user explicitly asks for "日报".
- Use `mode=selected` by default. Use `mode=all` only when the user explicitly asks for all/full/complete items.
- For company or topic queries, use server-side `q=<keyword>` instead of fetching a batch and filtering locally.
- For "最近 N 天 X", always set `since` to N days ago. The items endpoint should not be used beyond the latest 7 days.
- If the user asks for a specific AIHOT category, use the category parameter when helpful:
  - `ai-models`: AI 模型发布/更新
  - `ai-products`: AI 产品发布/更新
  - `industry`: AI 行业动态
  - `paper`: AI 论文研究
  - `tip`: AI 技巧与观点
- Fetch serially and avoid high-frequency polling. Treat cursor-like pagination tokens as opaque if the response includes them.
- Convert `publishedAt` from UTC ISO time into Beijing time and a human-readable form such as `今天上午 09:48`, `2 小时前`, `昨天`, or `5/7 00:43`. Do not show raw ISO timestamps to the user.
- Prefer `title` for Chinese output. Use `title_en` only when the user asks for English or `title` is empty.

## Event Types

Classify each news item into one primary event type:

- 模型发布: foundation model, reasoning model, multimodal model, embedding/reranker, voice/video model.
- 产品发布: AI assistant, coding tool, search, office tool, enterprise SaaS feature, consumer AI app.
- 算力硬件: GPU, ASIC, HBM, server, data center, networking, optical module, power and cooling.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paopao-zero/ai-industry-hot](https://github.com/paopao-zero/ai-industry-hot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
