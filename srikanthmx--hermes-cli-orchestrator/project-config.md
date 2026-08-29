---
trigger: always_on
description: **Read this before doing anything. Do not re-derive the project's purpose — it's here.**
---

# AGENTS.md — Hermes CLI Governor (`cli-orchestrator`)

**Read this before doing anything. Do not re-derive the project's purpose — it's here.**

---

## 0. North Star (the whole point)

This plugin turns Hermes into a **governance layer over many cheap/free brains and CLIs** so the agent runs at ~$0 **and never hits a wall.**

The central idea: **there is no single "best brain." The *pool* is the brain.**

- Consistency and learning in Hermes are **model-independent** — memory, skills, session/journey persist no matter which backend answered a given call. So the brain can change under the hood on every call and it's still *the same agent* with the same memory and behavior.
- Therefore the goal is **not** "pick Codex or Ollama." It's: **stack as many independent quota buckets as possible and rotate across them, auto-skipping exhausted ones, so the user never stalls and never notices the switch.**
- Each of these is a **separate quota bucket**, and they add up:
  - Subscription providers: Codex (ChatGPT), Copilot — premium but **harshly capped**.
  - Free OAuth: Qwen-OAuth (~2000/day), Nous.
  - Free API: Gemini API, OpenRouter `:free`, HuggingFace, Z.ai — each its own cap.
  - Trial credits: NVIDIA, Novita, GMI.
  - **CLI workers via `cli_delegate`**: agy, opencode, codex-exec — *more* separate quota.
  - **Multiple keys/accounts per provider** — multiplies each bucket.
  - Ollama — **optional bonus for users who have it. NEVER the assumed floor** (most users won't set it up).

The plugin "boasts of a consistent brain and a governance layer that hides the difference between switched models." Every design decision must serve that. If a suggestion relies on one model, or assumes Ollama, it's wrong.

---

## 1. Core mental model — do NOT confuse these

| Concept | What it is | Where it lives |
|---|---|---|
| **Model / provider (the "brain")** | an LLM endpoint Hermes calls to reason | `config.yaml` `model` + `fallback_providers`; needs OAuth or an API key |
| **CLI (a "worker")** | a subprocess the governor **delegates tasks to** (`cli_delegate` / `/cli-delegate`) | detected on PATH; NOT an LLM endpoint |

- A CLI (agy, opencode, …) **can never appear in the models list** and **can't be a cron's brain** — it's a worker. It can do heavy work *inside* a run via `cli_delegate`, but the orchestrating brain must be a model/provider.
- "Consistent brain" = the model-independent memory/skills/session, **not** any single model.
- **Hermes can already RUN CLIs** (native `terminal` tool; the desktop app has an integrated xterm terminal + an "agent terminal"). So *executing* a CLI is not the differentiator. Our value is **governing** that execution — cap-aware fallback across CLIs, priority, usage tracking, and a deterministic path that doesn't rely on the model choosing to emit a `terminal` call.

## 1b. Positioning vs Hermes v0.18 (native vs ours — keep claims honest)

Installed/latest Hermes is **v0.18.0**. It NATIVELY has, so **do NOT sell these as the plugin's**:
- **`moa`** (Mixture of Agents — multi model/provider slots), **`fallback`** (fallback provider chain), **`auth`** (pooled provider credentials = multi-account), **`proxy`** (OpenAI-compatible proxy to OAuth providers), **`cron`**.
- Model-independent **memory / skills / sessions / learning**.
- **Running CLIs** (terminal tool + desktop terminal/agent-terminal).
- A media-provider framework (edge-tts, faster-whisper, image/video gen plugins).

**Genuine differentiators (what to actually sell):**
1. **Extends governance from model *providers* to your local *CLIs*.** Hermes governs API/OAuth model backends; this brings codex/agy/opencode/claude/cursor/etc. into the same regime — detect, cap, delegate with fallback, track usage.
2. **`cli_delegate` / `/cli-delegate`** — deterministic, cap-aware, auto-fallback delegation across CLIs. The reliable path when a weak brain would otherwise *narrate* a delegation instead of running it.
3. **A single control-plane dashboard** for the whole fleet — CLIs + models + media in one place: status, caps, keys (+ get-key links), guided install, per-category routing. Hermes has scattered config subcommands and a terminal, not this unified governance UI.
4. **Ops/health tooling per backend** — live "Check sign-in" (real sample call), install stepper + "Ask AI for help" (answered via a governed CLI), provenance labels (verified/catalog/custom), per-CLI caps + usage.
5. **`generate_music`** tool (Hermes has no music framework).

One-liner: **"The CLI & backend control plane for Hermes — extends Hermes's model governance to your local AI CLIs, with one dashboard to run the whole fleet."**

---

## 2. Hard-won facts (do not relearn these the hard way)

- **Codex limit is brutal:** ~3 cron iterations exhaust it, then a **~28-day cooldown**. Codex is NOT a workhorse for crons. Never design around Codex as the steady brain.
- **Ollama is optional**, never assumed. Not everyone installs it.
- **Gemini CLI free tier is DEAD** ("IneligibleTierError — migrate to Antigravity"). The **Gemini API key** path still works; use that, not the CLI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srikanthmx/hermes-cli-orchestrator](https://github.com/srikanthmx/hermes-cli-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
