---
trigger: always_on
description: litelm is a 2,912 LOC reimplementation of litellm's core routing+formatting. Its DSPy contract and seven providers have been verified live; everything beyond the evidence below is untested.
---

# Ground Truth (updated 2026-09-19)

litelm is a 2,912 LOC reimplementation of litellm's core routing+formatting. Its DSPy contract and seven providers have been verified live; everything beyond the evidence below is untested.

## What's Actually Proven

- **DSPy integration:** All 7 DSPy execution paths work (Predict, CoT, typed signatures, streaming, embeddings, ReAct, multi-output). 10 live smoke tests, re-certified 2026-09-11.
- **7 providers verified live:** openai, anthropic, groq, mistral, xai, openrouter, azure. 45 live tests covering basic completion, streaming, streaming+usage, tool calls, streaming tool calls, embeddings, error mapping; all re-certified 2026-09-11.
- **265 own tests pass**, 55 skipped (live tests needing API keys), including a 2026-09-19 local gate run against the current lock.
- **Current classified ported baseline:** 75 tests pass against LiteLLM `9a715df2` from 2026-09-11. After explicit contract-based scope review, no remaining assertion/runtime failure is actionable. Raw counts are not comparable to the March baseline because upstream's test layout and conftest behavior changed substantially.

## What's NOT Proven

- **10 providers with no API keys:** bedrock, cloudflare, together_ai, fireworks_ai, deepseek, perplexity, deepinfra, gemini, cohere, ollama. They route through OpenAI-compat which works for the 7 tested providers, but provider-specific quirks (like Mistral's `type=None` tool calls) can only be found with live testing.
- **Bedrock + Cloudflare handlers:** Bedrock uses custom SigV4 auth; Cloudflare delegates to its OpenAI-compatible `/ai/v1` endpoint. Both still have zero live testing.

## Honest Ported Test Breakdown (2,121 collected, synced 2026-09-11)

The harness loads only litelm's shim conftest and excludes suites outside the declared routing/formatting contract. Upstream's repository growth means these counts are not comparable to the March baseline.

| Bucket | Count | What it means |
|--------|------:|---------------|
| Passed | 75 | Working |
| Assertion failures | 25 | Reviewed; LiteLLM-specific config, annotations, or prompt rewriting |
| Runtime errors | 68 | Reviewed; internal model registries/transforms or unsupported providers |
| Needs API key | 32 | Would need unavailable credentials |
| Skipped | 57 | Skipped by test logic |
| Import errors | 1,864 | Tests importing LiteLLM internals/features litelm intentionally does not implement |

The classifier reports **0 remaining high-relevance failures** after explicit path-level review. The audit found and fixed actionable gaps in future Claude thinking, Anthropic schema handling/native structured output, embedding retry propagation, and exception response headers. The categorized JUnit result was produced outside the repository. The source-level audit reviewed 360 core-path commits from `649eb2d` through `9a715df2`; 171 candidates received explicit decisions with no unknown rows.

## Project Plan

### Phase 1: Harden Core (complete) — litelm as litellm drop-in for DSPy

- Core routing, completion, streaming, embedding, text_completion, responses API
- 4 custom handlers (anthropic, bedrock, cloudflare, mistral)
- Own type system, exception hierarchy, error wrapping across all SDK paths
- DSPy contract fully satisfied across all 7 verified execution paths
- Client caching (thread-safe, Azure api_version-aware, Bedrock cached)
- All actionable ported test failures fixed: exception kwargs, `__getitem__`, images, kwarg stripping, mock streaming, mock_completion, n support
- No remaining actionable ported test failures (14 high-relevance failures all out of scope)
- Error mapping complete: NotFoundError, PermissionDeniedError, UnprocessableEntityError mapped in all 4 handlers
- `get_llm_provider()` exported for litellm compat

### Phase 2: dspy-lite — the actual goal

Fork DSPy, replace `import litellm` with `import litelm`, remove litellm dependency entirely. Strip proxy/router/caching/budgeting/etc. This is where the value is — litelm is just the enabler.

### Phase 3: More providers (as needed)

Only worth doing when a specific use case demands it. Each new provider key can uncover quirks like Mistral's `type=None`. Bedrock and Cloudflare handlers need live testing before they can be called proven.

### What's explicitly out of scope (and stays out)

Router, proxy, caching, budgeting, agents, guardrails, image gen, audio, OCR, fine-tuning, batches, assistants, scheduler, callback integration frameworks (opik, mlflow, etc), provider config registry, a2a protocol, compactifai. Minimal LiteLLM-compatible completion success/failure callback hooks are implemented.

## Architecture Comparison: litellm vs litelm (audited 2026-03-16)

### What litellm actually is

litellm is ~40K LOC across 1,667 Python files. Its public namespace exports **1,323 attributes** — ours exports 44. The difference is almost entirely feature sprawl that doesn't touch the core routing path.

**litellm's layers:**

| Layer | LOC | What it does | In litelm? |
|-------|-----|-------------|:---:|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kennethwolters/litelm](https://github.com/kennethwolters/litelm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
