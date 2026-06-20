---
trigger: always_on
description: >
---


# ZeroAPI v3.8.37 - Plugin-Based Model Routing

You are configuring an OpenClaw **gateway plugin**. ZeroAPI routes **eligible** messages at runtime through the `before_model_resolve` hook. You do **not** route messages manually. Your job is to inspect the user's setup, generate `zeroapi-config.json`, align `openclaw.json`, install/update the plugin, and verify the result.

This is **not prompt-based routing**. There is no extra LLM routing call, no context serialization, and no sub-agent layer in the runtime path.

## Channel-first contract

Treat `/zeroapi` as a **chat-native onboarding flow**. The primary surfaces are Slack, Telegram, WhatsApp, Matrix, Discord, terminal chat, and other OpenClaw text channels.

Behavior rules:

- ask **one short question at a time**
- prefer **numbered choices**
- keep replies compact and channel-safe
- do not dump full JSON or large benchmark tables into chat
- never ask the user to paste secrets into chat
- when host access is required, give the shortest safe command and then resume the chat wizard
- if the user first asks what the repo does or whether it is useful, answer that **neutrally from the repo/docs first**
- do **not** infer repo ownership from the GitHub owner name, old memory, or previous installs unless the user explicitly states ownership or you have just verified the live host state
- if the user shared the ZeroAPI GitHub link or asked a repo/product question, treat that as a **fresh-product explanation trigger** and do not mention local installs, MEMORY.md, or live host state in the first answer
- when the user says only `kuralım`, `install`, or similar right after that repo/product question, continue the **fresh install flow** instead of jumping to "already installed" unless they explicitly asked to inspect the live host

If the channel exposes only the generic skill runner, `/skill zeroapi` is an acceptable entry point. `scripts/first_run.ts` is only a terminal fallback for repo-local or shell-driven installs.

## Provider exclusions

ZeroAPI only routes across subscription-covered alternatives.

- **Anthropic (Claude):** as of 2026-04-04, Claude subscriptions no longer cover OpenClaw usage in third-party tools. Anthropic models should not be included in ZeroAPI routing.
- **Google (Gemini):** CLI OAuth usage with third-party tools is a ToS violation as of 2026-03-25. Google/Gemini should not be included in ZeroAPI routing.
- **xAI OAuth vs API keys:** OpenClaw 2026.5.20+ can use SuperGrok device-code OAuth through the native `xai` provider; older 2026.5.18+ installs use browser OAuth. Only enable `xai/*` models in a subscription policy when that runtime account is OAuth/subscription-backed. Do not auto-add plain `XAI_API_KEY` billing as subscription capacity.

## How it works

Three layers:

```text
Layer 1: benchmarks.json
  Embedded benchmark data maintained in the repo.

Layer 2: SKILL.md (/zeroapi runs once)
  Scans setup, chooses model pools, writes config, installs plugin.

Layer 3: Plugin runtime (before_model_resolve)
  Capability filter -> conservative classification -> balanced benchmark-aware selection.
```

Important authority order:

1. **OpenClaw runtime** (`openclaw.json`) is the runtime authority.
2. **ZeroAPI config** (`zeroapi-config.json`) is policy/config input for the plugin.
3. Plugin may suggest a `modelOverride` on eligible turns only.

ZeroAPI also supports a **subscription-aware foundation**:
- a fixed provider tier catalog
- a persistent global subscription profile
- an optional same-provider `subscription_inventory` for multi-account setups
- optional agent-level partial overrides
- benchmark-frontier candidate ordering without exposing private usage data

## Supported providers

Six subscription or account-quota providers are currently supported by the routing policy.

| Provider | OpenClaw ID | Auth | Tiers |
|----------|-------------|------|-------|
| OpenAI | `openai-codex` | OAuth PKCE via ChatGPT | Plus, Pro |
| Kimi | `moonshot` (`kimi`, `kimi-coding` legacy aliases) | API key | Moderato, Allegretto, Allegro, Vivace |
| Z AI (GLM) | `zai` | API key (`zai-coding-global`) | Lite, Pro, Max |
| MiniMax | `minimax-portal` (`minimax` alias) | OAuth portal | Starter, Plus, Max, Ultra-HS |
| Qwen Portal | `qwen-portal` (`qwen`, `qwen-dashscope` aliases) | OAuth portal | Free OAuth |
| xAI Grok OAuth | `xai` (`xai-oauth` legacy Hermes alias) | OpenClaw or Hermes OAuth via SuperGrok | SuperGrok |

See `references/cost-summary.md` for bundle examples and `references/subscription-catalog.md` for the public tier catalog used by the config.

## Task categories and routing basis

ZeroAPI classifies each eligible message into one of six categories, then picks the best available model for that category.

| Category | Primary Basis | Secondary | Typical Keywords |
|----------|---------------|-----------|------------------|
| Code | `coding_index` (reweighted) | `terminalbench` | implement, function, class, refactor, fix, test, debug, diff |
| Research | `gpqa`, `hle` | `lcr`, `scicode` | research, analyze, explain, compare, investigate |
| Orchestration | `0.6*tau2 + 0.4*ifbench` | — | orchestrate, coordinate, pipeline, workflow, parallel |
| Math | `math_index` | `aime_25` | calculate, solve, proof, optimize, formula |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dorukardahan/ZeroAPI](https://github.com/dorukardahan/ZeroAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
