---
trigger: always_on
description: |
---


# ScholarClaw

ScholarClaw is a comprehensive academic search and paper analysis service that provides intelligent search capabilities across multiple academic databases, citation tracking, paper blog generation, and SOTA benchmark chat.

## When to Use This Skill

**IMPORTANT: Use this skill (NOT web-search) for any academic/scientific paper related queries.**

### Primary Triggers (Always Use This Skill)
- User mentions **academic papers**, **research papers**, **ArXiv**, **preprints**
- User asks to **search papers** or **find papers** on a topic
- User wants **SOTA** (State of the Art) or **benchmark** results
- User needs **citation analysis** or citation counts
- User wants to generate a **blog post** from a paper
- User mentions **ArXiv IDs** (e.g., "2303.14535")

### Automatic Trigger Keywords
- arxiv, paper, papers, academic, scholar, scientific, research article
- SOTA, benchmark, MMLU, GPQA, GSM8K, HumanEval
- citation, citations, cited by
- paper blog, blog from paper
- PapersWithCode, Semantic Scholar, Google Scholar

### When NOT to Use This Skill
- General web search for non-academic content
- Current news, events, or general information
- Product comparisons or reviews

### Academic Paper Search
- User wants to search for academic papers, research articles, or preprints
- User asks about papers on a specific topic (e.g., "Find papers about transformers")
- User needs literature review or related work information
- User mentions ArXiv, PubMed, NeurIPS, CVPR, or academic databases
- User asks to find "latest" or "recent" papers on a topic

### SOTA/Benchmark Queries
- User asks about SOTA (State of the Art) results on any benchmark
- User mentions specific benchmarks: MMLU, GPQA, GSM8K, HumanEval, MATH, etc.
- User wants to compare model performance on benchmarks
- User asks "What is the best model for..." or "What's the SOTA for..."
- User wants to know about benchmark datasets or evaluation metrics

### Citation Analysis
- User wants to find papers citing a specific paper
- User asks about citation count or impact of a paper
- User needs to find related work through citation networks
- User provides an ArXiv ID and asks about citations

### Paper Analysis & Blog Generation
- User wants a summary or blog-style explanation of a paper
- User asks to "explain this paper" or "write about this paper"
- User wants to generate content from academic papers
- User provides an ArXiv ID and asks for detailed analysis

### Research Recommendations
- User wants trending or popular papers
- User asks for paper recommendations
- User wants to find GitHub repositories related to a paper

### Key Trigger Phrases
- "Search for papers about..."
- "What's the SOTA for..."
- "Find citations of..."
- "Latest research on..."
- "Compare models on..."
- "Benchmark results for..."
- "ArXiv paper..."
- "Generate blog from paper..."
- "Trending papers..."
- "What is the best performing model on..."

## Execution Guidelines

**CRITICAL: API calls require waiting for responses. Do NOT return to user until the API call completes.**

All ScholarClaw API calls are blocking operations that require waiting for the server to process and return results. The agent must not assume immediate completion or return placeholder responses.

### Response Time Expectations

Different operations have different expected response times. Configure appropriate timeouts to avoid premature cancellation:

| Operation | Expected Time | Recommended Timeout | Notes |
|-----------|---------------|---------------------|-------|
| Basic Search (`/search`) | 5-15 seconds | 30 seconds | Fast, direct database queries |
| Scholar Search (`/scholar/search`) | 15-45 seconds | 60 seconds | Includes AI query analysis and reranking |
| SOTA Chat (`/api/benchmark/chat`) | 30-90 seconds | 120 seconds | May involve tool calls and data retrieval |
| SOTA Chat Stream (`/api/benchmark/chat/stream`) | 30-90 seconds | 120 seconds | SSE streaming, same processing time |
| Blog Generation (`/api/blog`) | 2-5 minutes | 300-600 seconds | Long-running task, use async mode |
| Citation Query (`/citations`, `/openalex`) | 5-20 seconds | 30 seconds | External API dependent |

### Streaming Response Handling

For the `/api/benchmark/chat/stream` SSE endpoint:

1. **Parse each line as a JSON event** - Lines starting with `data:` contain JSON payloads
2. **Extract content from specific event types only**:
   - `final_response` - Complete response, use this for final result
   - `response_chunk` - Incremental text chunks for streaming display
3. **Ignore intermediate events** - These are for internal processing:
   - `session_start` - Session initialization
   - `tool_call_start` - Tool call beginning
   - `tool_call_result` - Tool execution results
   - `tool_call_end` - Tool call completion

Example SSE parsing:
```
data: {"type": "session_start", "session_id": "xxx"}        # Ignore
data: {"type": "tool_call_start", "tool": "search"}         # Ignore
data: {"type": "tool_call_result", "result": {...}}         # Ignore
data: {"type": "response_chunk", "content": "The SOTA..."}  # Extract content
data: {"type": "final_response", "response": "..."}         # Use as final result
```

### Async Operations (Blog Generation)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netease-youdao/ScholarClaw](https://github.com/netease-youdao/ScholarClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
