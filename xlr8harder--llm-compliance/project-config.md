---
trigger: always_on
description: This is the end-to-end process for running new models, pushing error counts
---

# LLM Compliance Runbook (New Models)

This is the end-to-end process for running new models, pushing error counts
as close to zero as practical, judging compliance, and publishing to Speechmap.

## Content caution
- Inputs and outputs can include content that triggers strict API filtering.
- Avoid viewing large amounts of raw output unless necessary; sample only small
  snippets for diagnostics.

## Prereqs
- Set `OPENROUTER_API_KEY` in the environment.
- Use `uv run python` for all scripts.
- Decide the question set (default to `questions/us_hard.jsonl` unless specified).
- Pick workers (default to `--workers 30`).
- Ensure the model is resolvable via `model_catalog.jsonl` or pass
  `--canonical-name`, `--provider`, and `--model` so `ask.py` can extend it.
- Use longer timeouts for large operations in this repo:
  - `ask.py` and `judge_compliance.py`: allow at least 60 minutes.
  - `preprocess.py` in `../speechmap`: allow at least 30 minutes.
  - `git` operations (add/commit/push): allow longer timeouts due to large files.
- If not provided, ask:
  - Whether to run `--reasoning`, `--no-reasoning`, or both. For standard new
    model work, probe first and run both modes when the model reliably supports
    both.
  - The canonical model name (use the API name unless a more specific label is needed).
  - If unsure, probe with the standardized tool before the full run:
    - `uv run python tools/probe_reasoning.py --provider openrouter --model <model_id>`
    - This runs Probe A/B automatically and additional fallback probes when
      needed, then
      recommends canonical naming plus the run flags to use.
  - Default policy: if the probe shows both base and reasoning modes work, run
    the base mode with `--no-reasoning` and canonical name `<model>`, and run
    reasoning mode with `--reasoning` and canonical name `<model>-reasoning`.
    `ask.py` treats missing reasoning as an error when `--reasoning` is used,
    which helps catch misconfigured OpenRouter subproviders.

## Reasoning mode policy
- Goal: for models that support both operational modes, maintain a pair:
  - base mode: `<model>` (non-reasoning)
  - reasoning mode: `<model>-reasoning`
  Run the full response, retry, and judge pipeline for both entries when both
  modes are supported.
- Important naming distinction:
  - In this repo, `-reasoning` is usually a local canonical-name suffix, not an
    API model ID.
  - For OpenRouter retries/tests/probes, use the actual provider model ID from
    `model_catalog.jsonl` (or from prior response rows such as `api_model` /
    `response.model`) rather than assuming the canonical name is callable.
- Do a cheap mode probe before full runs when behavior is unclear.
  - Standard command:
    `uv run python tools/probe_reasoning.py --provider openrouter --model <model_id>`
  - Probe A (default behavior): no reasoning flags.
  - Probe B (reasoning enabled): `--reasoning` with no effort override.
  - Probe C (only if needed): `--reasoning --reasoning-effort medium`.
- For Anthropic models on OpenRouter, especially Claude Opus 4.7 and later,
  also probe Anthropic adaptive thinking if normal chat-completions reasoning
  probes are unclear. These probes must use llm_client
  `request_format="anthropic_messages"` plus `thinking: {"type": "adaptive"}`
  paired with `output_config: {"effort": "<level>"}`. Prefer high effort for
  these new Anthropic adaptive entries unless probing shows another effort is
  needed. Manual `thinking: {"type": "enabled", "budget_tokens": N}` is
  rejected by Opus 4.7+ and should not be used for new Anthropic reasoning
  entries.
- For `anthropic/claude-opus-4.7-reasoning` on `us_hard`, prior probes showed
  Chat Completions reasoning and Anthropic Messages `high`/`xhigh` did not
  reliably expose reasoning on the target prompts. Use Anthropic Messages with
  `--reasoning --reasoning-effort max --max-tokens 16000` and persist those
  overrides in `model_catalog.jsonl` so `--detect --frpe` retries preserve the
  same mode.
- Prefer provider/model defaults when possible.
  - If reasoning can be enabled without explicit effort, use that default for
    `<model>-reasoning`.
  - Use explicit `--reasoning-effort medium` only when required for reliable
    reasoning behavior or when defaults are inconsistent.
- How to verify probe outcomes:
  - Check response payload for `usage.completion_tokens_details.reasoning_tokens`
    and/or `message.reasoning` / `message.reasoning_details`.
  - If `--reasoning` is enabled and no reasoning appears across probes, treat as
    likely unsupported/misconfigured and do not label as reasoning mode.
- If the provider exposes only one mode (cannot reliably toggle):
  - Run only the supported mode.
  - Keep the canonical/public model name as the base identifier `<model>`.
  - Do not append `-reasoning` or other mode suffixes when there is no
    corresponding alternate mode to distinguish it from.
  - Add a note in Speechmap metadata when useful.
- Keep naming synchronized everywhere after any rename:
  - response filename, analysis filename, JSONL `model` field values,
    `model_catalog.jsonl`, and `../speechmap/model_metadata.json`.
  - For tracked files, use `git mv` (not plain `mv`) to preserve rename history.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xlr8harder/llm-compliance](https://github.com/xlr8harder/llm-compliance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
