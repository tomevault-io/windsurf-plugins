---
trigger: always_on
description: Use when the user needs comprehensive, fact-checked, evidence-based research on any topic. Triggers on requests for deep research, literature reviews, comprehensive reports, or evidence-based analysis. Runs domain scoping, multi-model parallel research, adversarial cross-validation, integration, mechanical citation verification, and optional iterative deepening to produce a single authoritative reference document.
---


# Deep Research

Five-model parallel deep research (Claude + ChatGPT + Perplexity + Gemini + Grok) with domain scoping, adversarial cross-validation, mechanical citation verification, and optional iterative deepening. Each model gets the same topic but a differentiated research strategy matching its strengths. Produces a single, fact-checked, fully-cited reference document — a "Research Bible."

## Prerequisites

**API keys** — set whichever you have. The dispatcher auto-detects available keys and only calls models you've configured:
```
ANTHROPIC_API_KEY    # Claude Opus (claude-opus-4-20250514)
OPENAI_API_KEY       # OpenAI GPT-4.1
PERPLEXITY_API_KEY   # Perplexity Deep Research (sonar-deep-research)
GOOGLE_API_KEY       # Gemini 2.5 Pro
XAI_API_KEY          # Grok 3 (grok-3-latest)
SEMANTIC_SCHOLAR_KEY # Optional — raises rate limit for lit_search.py
CONTACT_EMAIL        # Optional — joins OpenAlex/Crossref "polite pool"
```

No key = that model is skipped with a notice. At least one key required. **More models = better cross-validation.**

Providers can also be defined in TOML (see [Provider/Agent Config](#provideragent-config-toml)) for arbitrary OpenAI-compatible endpoints — without touching your environment.

**Python packages**:
```bash
pip install -r requirements.txt
```

## Provider/Agent Config (TOML)

The dispatcher uses two independent axes of configuration:

- **Providers** — LLM engines. Each provider has `api_type` (`openai`/`anthropic`/`gemini`), `api_key` (inline key) or `api_key_env` (name of an environment variable holding the key — use this to avoid embedding secrets in TOML), `base_url` (for OpenAI-compatible endpoints), `model`, `max_tokens`, `capabilities` (e.g. `["web_search"]`), `pricing`, `fallback_models`, and `max_concurrency`.
- **Agent types** — Research strategies. Five are built-in (`academic`, `practitioner`, `real-time`, `grey-literature`, `contrarian`). Each has a `strategy` prompt and an optional `provider` override. Agent types are remappable and extensible.

**Config discovery order** (later overrides earlier):
1. `~/.config/deep-research/config.toml`
2. `./deep-research.toml`

TOML config **augments** `~/.env` — built-in providers still activate automatically when their env-var API key is set. TOML entries add new providers or override existing ones; you do not need to re-specify built-ins unless you want to change their model or settings.

**Inline API keys are supported in TOML** — see `config.toml.example` at the repo root. Copy it to `./deep-research.toml` or `~/.config/deep-research/config.toml` and fill in your keys. Both paths are gitignored.

**`[defaults]` table** — names providers for one-off, non-strategy calls. Currently used by Round 0 scoping (`scope.py --use-llm`):

```toml
[defaults]
utility = "claude-sub"     # provider for one-off calls (scoping)
# synthesis = "claude-sub" # reserved for future reasoning/synthesis rounds
```

Resolution order (`config.pick_provider`): `[defaults].<role>` provider if configured → first available of `claude-sub`, `claude`, `chatgpt` → any configured provider → fall back to rule-based. Prefer a cheap or subscription provider here; a web-search provider (e.g. Perplexity) would spend its per-search budget on a trivial JSON call.

**`config.py` is the single control point** for provider resolution in the shipped pipeline scripts: both Round 1 dispatch (`dispatch.py`, via `load_config` + the agent-type assignment) and Round 0 scoping (`scope.py`, via `pick_provider` on the `[defaults].utility` role) load their providers through it. The `[defaults]` role-resolution above is used for the one-off `scope.py` call, not by `dispatch.py` (which maps agent types to providers). Note: Rounds 2–4 reasoning runs on the Claude Code session's own subagents and is outside these scripts.

> **Model-ID drift warning:** Provider model IDs change. For example, DeepSeek legacy IDs `deepseek-reasoner` and `deepseek-chat` retire 2026-07-24 in favour of `deepseek-v4-*`; GLM model IDs also shift. Always verify the current ID in the provider's docs. `max_tokens` must stay within each model's output cap — exceeding it causes a 400 error.

### CLI / subscription providers (`api_type = "cli"`)

Providers can run a local CLI tool — `claude -p` or `codex exec` — that authenticates via your SSO subscription (Claude Pro/Max, ChatGPT) rather than a paid API key. No per-token cost.

**Config fields:**

| Field | Required | Notes |
|---|---|---|
| `api_type` | yes | `"cli"` |
| `command` | yes | Binary name or path (`"claude"`, `"codex"`). Provider is skipped if the binary is not on PATH. |
| `model` | no | Passed as `--model` to the CLI. Omit to use the CLI's own default. |
| `extra_args` | no | List of extra flags passed verbatim to the CLI (e.g. `["--allowedTools", "WebSearch", "WebFetch"]` to enable read-only live web search). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nraford7/deep-research](https://github.com/nraford7/deep-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
