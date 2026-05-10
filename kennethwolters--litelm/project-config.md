---
trigger: always_on
description: litelm is a 2,660 LOC reimplementation of litellm's core routing+formatting. It works as a **DSPy backend for 6 providers** — that is the only verified claim. Everything beyond that is untested.
---

# Ground Truth (updated 2026-04-17)

litelm is a 2,660 LOC reimplementation of litellm's core routing+formatting. It works as a **DSPy backend for 6 providers** — that is the only verified claim. Everything beyond that is untested.

## What's Actually Proven

- **DSPy integration:** All 7 DSPy execution paths work (Predict, CoT, typed signatures, streaming, embeddings, ReAct, multi-output). 10 live smoke tests.
- **7 providers verified live:** openai, anthropic, groq, mistral, xai, openrouter, azure. 44 live tests covering basic completion, streaming, streaming+usage, tool calls, streaming tool calls, embeddings, error mapping.
- **161 own tests pass**, 47 skipped (live tests needing API keys).
- **65 of litellm's ported tests pass** out of 79 high-relevance tests (82.3%). The other 1060+ collected tests fail at import — they reference litellm internals (Router, proxy, provider-specific LLM modules) we intentionally don't implement. (Upstream re-synced 2026-03-16; test count changed due to litellm restructuring.)

## What's NOT Proven

- **10 providers with no API keys:** bedrock, cloudflare, together_ai, fireworks_ai, deepseek, perplexity, deepinfra, gemini, cohere, ollama. They route through OpenAI-compat which works for the 7 tested providers, but provider-specific quirks (like Mistral's `type=None` tool calls) can only be found with live testing.
- **Bedrock + Cloudflare handlers:** 420 LOC of custom handler code (SigV4 auth, raw httpx SSE parsing) with zero live testing. The Bedrock client cache was just added — also untested against a real endpoint.

## Honest Ported Test Breakdown (1,080 collected, synced 2026-03-16)

After fixing conftest to exclude nested dirs by basename (not just root-level glob):

| Bucket | Count | What it means |
|--------|------:|---------------|
| Passed | 56 | Working |
| Assertion failures | 12 | Our code runs but produces wrong results |
| Runtime errors | 6 | Our code runs but crashes |
| Needs API key | 29 | Would need live credentials |
| Skipped | 70 | Skipped by test logic |
| Timeout | 1 | Timed out |
| Import errors | 906 | Tests importing litellm internals we don't implement — **will never pass** |

Of the 14 high-relevance failures, **none are actionable** — all are out of scope (compactifai 7, fallbacks/Router 3, gemini no_api_key 2, poetry deps 1, Anthropic system message format 1). 0 regressions vs previous baseline; 9 fewer passes are tests removed upstream.

## Project Plan

### Phase 1: Harden Core (complete) — litelm as litellm drop-in for DSPy

- Core routing, completion, streaming, embedding, text_completion, responses API
- 4 custom handlers (anthropic, bedrock, cloudflare, mistral)
- Own type system, exception hierarchy, error wrapping across all SDK paths
- DSPy contract fully satisfied for 6 providers
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

Router, proxy, caching, budgeting, agents, guardrails, image gen, audio, OCR, fine-tuning, batches, assistants, scheduler, callbacks/integrations (opik, mlflow, etc), provider config registry, a2a protocol, compactifai.

## Architecture Comparison: litellm vs litelm (audited 2026-03-16)

### What litellm actually is

litellm is ~40K LOC across 1,667 Python files. Its public namespace exports **1,323 attributes** — ours exports 44. The difference is almost entirely feature sprawl that doesn't touch the core routing path.

**litellm's layers:**

| Layer | LOC | What it does | In litelm? |
|-------|-----|-------------|:---:|
| `main.py` — core completion engine | 7,601 | `completion()`, `acompletion()`, `embedding()`, streaming, error handling | **Yes** (300 LOC) |
| `utils.py` — helpers | 9,313 | Token counters, param validation, model info lookups, `get_optional_params()` | **No** |
| `router.py` — load balancer | 9,611 | Fallback routing, round-robin, cost-based, TPM/RPM strategies | **No** |
| `cost_calculator.py` — pricing | 2,253 | `completion_cost()` from 500KB model pricing JSON | **No** |
| `llms/` — 90+ provider handlers | ~15K | Per-provider translation classes | **4 handlers** (1,135 LOC) |
| `types/` — 156 type files | ~8K | Response types, provider-specific types, proxy types | **1 file** (361 LOC) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kennethwolters/litelm](https://github.com/kennethwolters/litelm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
