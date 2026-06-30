---
trigger: always_on
description: The `GeminiAdapter` calls Google's Generative Language API. effGen ships
---

# Gemini & Gemma in effGen

The `GeminiAdapter` calls Google's Generative Language API. effGen ships
a small registry of free / cheap text-out models so you can pick one
without leaving the framework.

## Quick start

```python
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.models.base import GenerationConfig

model = GeminiAdapter(model_name="gemini-3.1-flash-lite")
model.load()
result = model.generate(
    "Summarise the second law of thermodynamics in one sentence.",
    config=GenerationConfig(max_tokens=64, temperature=0.2),
)
print(result.text)
```

The adapter reads `GOOGLE_API_KEY` from `~/.effgen/.env` (or the project's
`.env`) automatically when `python-dotenv` has been initialized; you can
also pass `api_key=...` explicitly.

## Recommended models (text-out only)

```python
import effgen

# Models that are reliably callable on the free tier today.
for m in effgen.gemini_recommended_models(tier="free"):
    print(m["id"], m["family"], "RPM=", m["rpm"], "RPD=", m["rpd"])

# All registered models (free + premium):
effgen.gemini_recommended_models(tier="all")

# Inspect a single model:
effgen.gemini_model_info("gemini-3.1-flash-lite")  # alias resolves
```

| Model                                | Family       | Tier    | RPM | TPM     | RPD    | Tools | Thinking | Grounding |
|--------------------------------------|--------------|---------|-----|---------|--------|-------|----------|-----------|
| `gemini-3.1-flash-lite`              | flash-lite   | free    | 15  | 250 K   | 500    | yes   | yes      | no¹       |
| `gemini-3-flash-preview`             | flash        | free    | 5   | 250 K   | 20     | yes   | yes      | yes       |
| `gemini-3-pro-preview`               | pro          | premium | —   | —       | —      | yes   | yes      | yes       |
| `gemini-3.1-pro-preview`             | pro          | premium | —   | —       | —      | yes   | yes      | yes       |
| `gemini-2.5-flash-lite`              | flash-lite   | free    | 10  | 250 K   | 20     | yes   | no       | yes       |
| `gemini-2.5-flash`                   | flash        | free    | 5   | 250 K   | 20     | yes   | no       | yes       |
| `gemini-2.5-pro`                     | pro          | premium | —   | —       | —      | yes   | yes      | yes       |
| `gemini-2.0-flash`                   | flash        | premium | —   | —       | —      | yes   | no       | yes       |
| `gemini-2.0-flash-lite`              | flash-lite   | premium | —   | —       | —      | yes   | no       | yes       |
| `gemma-4-26b-a4b-it` / `gemma-4-31b-it` | gemma     | free    | 15  | unlim.  | 1 500  | no    | no       | no        |

¹ Google Search grounding hits quota on the free-tier for `gemini-3.1-flash-lite`. Use `gemini-2.5-flash` or higher for grounding.

Limits reflect Google's free-tier defaults as of 2026-04-25 — check
[ai.google.dev/gemini-api/docs/rate-limits](https://ai.google.dev/gemini-api/docs/rate-limits)
for the current numbers.

## Thinking budget (extended reasoning)

Gemini 2.5-Pro and the full Gemini 3.x family support an optional internal
reasoning step ("thinking") before producing an answer. You can control it
via two `GenerationConfig` fields:

| Field | Type | Default | Meaning |
|---|---|---|---|
| `thinking_budget` | `int \| None` | `None` | `None` = field not sent (model decides). `0` = disabled. `> 0` = token budget for reasoning. |
| `include_thoughts` | `bool` | `False` | When `True`, the reasoning trace is surfaced in `result.metadata["thinking"]`. |

```python
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.models.base import GenerationConfig

model = GeminiAdapter(model_name="gemini-3.1-flash-lite")
model.load()

result = model.generate(
    "A train travels 120 km in 1.5 hours. What is its average speed?",
    config=GenerationConfig(
        thinking_budget=8192,
        include_thoughts=True,
        max_tokens=512,
    ),
)
print("Thinking:", result.metadata.get("thinking", "(none)"))
print("Answer:", result.text)
print("Thinking tokens:", result.metadata["thoughts_token_count"])
```

**Notes:**
- Models that don't support thinking (Gemma, Gemini 2.5-Flash-Lite, Gemini 2.0)
  silently ignore `thinking_budget`. No error is raised.
- `include_thoughts=False` (default) still uses the thinking budget internally
  but doesn't return the trace, saving output tokens.
- The thinking trace can be long. Set `include_thoughts=True` only when you
  need to inspect reasoning.

### Models with thinking support

| Model | Thinking | Notes |
|---|---|---|
| `gemini-3.1-flash-lite` | yes | Free tier, 15 RPM / 500 RPD |
| `gemini-3-flash-preview` | yes | Free tier, 5 RPM / 20 RPD |
| `gemini-3-pro-preview` | yes | Paid only |
| `gemini-2.5-pro` | yes | Paid only |
| `gemini-2.5-flash` | no | Free tier |
| `gemini-2.5-flash-lite` | no | Free tier |
| `gemma-*` | no | Open weights, generous free-tier RPD |

## Google Search grounding

When `GenerationConfig.grounding=True` and the model supports it, the adapter
activates Gemini's built-in Google Search tool. The model fetches live web
results and attributes its answer to real URLs returned in
`result.metadata["grounding_chunks"]`.

```python
from effgen.models.gemini_adapter import GeminiAdapter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ctrl-gaurav/effGen](https://github.com/ctrl-gaurav/effGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
