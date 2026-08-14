---
trigger: always_on
description: Invarail uses a **Router + Specialist** pattern with a **tool-loop (ReAct) engine** and **deterministic pipelines**.
---

# CLAUDE.md — Invarail AI Code Generation Guidelines

## Architecture

Invarail uses a **Router + Specialist** pattern with a **tool-loop (ReAct) engine** and **deterministic pipelines**.

```
Channel (Discord/Telegram/Slack/Web/Gmail/WhatsApp/MS Graph/iMessage/Chrome Extension)
  -> Router (phi4:14b, classifies intent into one category)
    -> Pipeline (deterministic stages — most categories)
    -> OR Specialist (config-assigned model + ReAct tool-loop — chat, config, personal)
      -> Tool Executor (sandboxed via Docker or allowlist)
        -> Response back to channel
```

**Inference backends (additive multi-backend):** A `MultiBackendClient` (`src/ollama/multi-backend.ts`, extends `OllamaClient`) routes each `chat`/`chatStream` call by model id. Foreground reasoning models (currently DeepSeek-V4-Flash — the swappable foreground slot) route to an **OpenAI-compatible ds4/DwarfStar** endpoint (github.com/antirez/ds4 — direct, NOT behind the gateway; launched without --think/--nothink → default thinking, high effort) via `OpenAICompatClient` (`src/ollama/openai-client.ts`); everything else (router phi4, NER phi4-mini, embedding, vision qwen3.6:27b) stays on the **Ollama gateway**. `embed()` always uses Ollama. Configured via `inference.backends[]` in config. The OpenAI client translates Ollama↔OpenAI shapes: `options.*`→top-level params (reserving reasoning headroom on `max_tokens` so a small `num_predict` can't starve a reasoning model into an empty completion), tool-call `arguments` string→object, `tool_call_id` stitching, SSE streaming, `usage`→`eval_count`/`prompt_eval_count`. **`think` control** (2026-08 eval): `OllamaChatParams.think` + per-specialist `think` config flows through dispatch→engine; the OpenAI-compat path forwards it only for backends declaring `supportsThink` (ds4 does, verified) and warn-once-omits otherwise — never a silent drop. Purely additive — the Ollama path is unchanged.

**Key components:**
- **Router** — phi4:14b, single-word classification into categories: `chat`, `web_search`, `memory`, `exec`, `cron`, `message`, `website`, `multi`, `config`, `task`, `research`, `personal`. Pre-model overrides for high-confidence patterns (PDF reports, calendar queries; bare URLs → website — a URL inside a larger request does NOT hijack routing). Model output is enum-grammar-constrained via `format` when the backend supports it. `config.router.timeout` is ENFORCED (Promise race → keyword fallback; the client's connection-retry loop no longer stalls messages past the budget). Fallback to `defaultCategory` on timeout/parse failure. Implemented in `src/router/classifier.ts`.
- **Pipeline engine** — `src/pipeline/executor.ts`. Deterministic stage-based workflows: extract, tool, parallel_tool, llm, code, branch, llm_branch, loop. Most categories use pipelines instead of letting the model decide the workflow. Extraction (`src/pipeline/extractor.ts`) uses grammar-constrained decoding (`format` JSON schema, auto-fallback if the backend rejects it), a 2048-token default budget (thinking counts against `num_predict` — 256 starved thinking models into emitting reasoning prose with no JSON; 2026-08 eval), `stripThinkingTags` before parsing (Qwen `<think>` AND Gemma-4 formats), JSON5-tolerant parsing, post-parse required/enum/coercion validation feeding the repair prompt, and best-effort params over aborting; `ExtractStage.fallback(ctx)` provides deterministic degrade-not-abort per stage. `llm_branch` output is enum-constrained.
- **Plan pipeline** — `src/pipeline/definitions/plan.ts`. LLM decomposes goals into specialist sub-tasks, self-reflects, executes via foreman handoffs with write-through artifacts. Used by `multi` category. The skills system was retired 2026-08-10 (successor: graph experience memory — experience informs execution, never expands authority; see DECISIONS).
- **Research pipeline** — `src/pipeline/definitions/research.ts`. [flow_gather] → decompose → per-facet parallel search + fetch + synthesis → analytical markdown report → **evidence verification** → deterministic markdown→HTML→PDF render with charts (absolute img paths — LibreOffice resolves relative src against the temp HTML's dir). **Flow-first gathering:** when the request EXPLICITLY names an available flow tool, `flow_gather` calls it once; `parseFlowGather` turns its `##` sections into facets and links into per-facet source pools; decompose/parse_angles skip (`when` gates); `researchAngle(ctx, angle, presetUrls)` fetches/synthesizes identically so verification works unchanged. Flow failure degrades to normal search. Strict naming only — NO semantic flow-matching (that's the skill-hijack bug class one layer up).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PeterGreenAppliedAI/Invarail](https://github.com/PeterGreenAppliedAI/Invarail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
