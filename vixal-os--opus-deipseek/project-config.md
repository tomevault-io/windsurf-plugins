---
trigger: always_on
description: Single-file (`bot.py`) multi-model Discord bot. EVA/MAGI-themed heuristic router over
---

# CLAUDE.md — Hydra (Opus-Deipseek)

Single-file (`bot.py`) multi-model Discord bot. EVA/MAGI-themed heuristic router over
**Claude (Balthasar) · DeepSeek (Melchior) · Gemini (Caspar)** plus open-weight heads
**Qwen (Rei) · GLM (Asuka)** on Fireworks and **Mistral (Mari)** on its own EU API. Two-tier memory,
prompt caching, bookclub mode, Mandarin (`!speak`) + French (`!french`) TTS,
research panel (`!research`), simulator mode (`!dummy` — base-model transcript completion),
and cost + carbon tracking.

This file is the working status + remaining-work tracker for the "Hydra Restructure"
spec. The full spec lives outside the repo (Sarah's Claude.ai artifact); this file is
self-contained — you don't need it to pick up the remaining work below.

## Run / verify
- `python bot.py` — needs `.env` (`DISCORD_TOKEN` + ≥1 model key) and `config.json`
  (`allowed_channels`, `default_model`).
- Syntax gate (no deps): `python -c "import ast; ast.parse(open('bot.py',encoding='utf-8').read())"`
- Import/logic test: `python -c "import bot"` works (deps installed). On Windows console
  prefix `PYTHONIOENCODING=utf-8` when printing bot output (emoji/CJK/IPA).

## Editing rules (load-bearing — read before touching bot.py)
- **Graceful degradation:** a missing API key disables ONLY that provider. The gating key follows
  the *active backend* (`provider.api_key_env`): by default `FIREWORKS_API_KEY` gates Qwen+GLM
  together and **Mistral has its own `MISTRAL_API_KEY`** (`api.mistral.ai`) — its flagship (Large 3)
  is on-demand-only on Fireworks, not serverless. `self_hosted`/`vertex` backends have no key and aren't
  key-gated (operator owns the local server / GCP ADC). Absent keys disable exactly those providers.
- **Config-driven registry (Phase 0).** `ProviderRegistry.from_config()` (built once in `__init__`
  from `config.json`'s optional `providers` block) wires every provider's client + backend. The 6
  `ModelProvider(...)` constants are the code-side default spec (pricing/energy/quirks +
  `sdk_type`/`api_key_env`/`base_url`/`backend`/`backends`); config supplies operator overrides
  (`enabled`/`backend`/`model`/`routing_tags`) + the `platform` toggle. **No `providers` block ⇒
  byte-for-byte the old behavior.** `routing_tags` is present but INERT (routing stays in
  `_estimate_confidence`).
- **`provider.name` is the canonical routing key.** `_select_model`, `_estimate_confidence`,
  `panel_members`, cost, and persistence all branch on it. NEVER rename it. (`provider.id` — the
  lowercase config key — is separate; it keys `config.json` + `self.clients`.) Display names and
  command aliases are a separate theme layer on top (see naming below).
- **Dispatch** routes on `provider.sdk_type` (`anthropic` | `openai_compatible` | `gemini`), set by
  the registry. Claude → anthropic SDK; Gemini bookclub → native `cachedContents` (or Vertex
  `CachedContent` when `backend="vertex"`); everything else → the OpenAI-compatible shim
  (`_generate_openai_compatible_response`) via `self.clients[provider.id]`. Per-provider wiring lives
  in the registry, not `__init__`.
- **Model slugs drift** — verify in the live libraries. Fireworks (verified 2026-06):
  `qwen3p7-plus`, `glm-5p2`, `deepseek-v4-pro` (under `accounts/fireworks/models/`). Mistral on
  its own API uses `mistral-large-latest` (→ Mistral Large 3, released 2025-12, 675B/41B MoE,
  Apache-2.0; priced $0.50/$1.50 per Mtok as of 2026-06 — a ~4× drop from Large 2). ⚠️ Large 3 is
  on Fireworks only as **on-demand/dedicated** (`mistral-large-3-fp8`), NOT serverless — so the
  own-API route stays correct AND is now the cheap one, not just the green one.
- New-provider **pricing + energy constants are estimates** flagged `VERIFY` in comments —
  confirm against the Fireworks pricing page before trusting `!cost` $ figures.

## Naming theme — now a toggleable **theme layer** (`config.json` top-level `"theme"`)
Themes are a **display-only skin**: `Flavor`/`Theme`/`THEMES` (just after `SIM_PROVIDER`) map each
`provider.id` → themed display name + command aliases + a persona note. Read in
`ProviderRegistry.from_config` (mirrors the `platform` toggle), applied onto `provider.display_name`
and the `self.alias_to_flag` map built in `__init__`. **The canonical `provider.name`, the
`**[Claude]**` label, and `MODEL_LABEL_NAMES` never change** — a Judah/Gold-Head turn is still
labeled `[Claude]` on the wire. Canonical bare prefixes (`!claude`/`!deepseek`/…) + `!think` work in
every theme; the theme only *adds* its flavor aliases. Three sets ship (default **`eva`** ⇒
byte-for-byte the old behavior):
- **`eva` — EVA/MAGI** (default). Trinity Balthasar/Melchior/Caspar; pilots Mistral=Mari (`!mari`),
  Qwen=Rei (`!rei`), GLM=Asuka (`!asuka`); sim=Dummy Plug (`!dummy`).
- **`isaic` — ISAIC** ("International System of AI Coopertition"), the twelve tribes: Judah=Claude
  (`!judah`), Joseph=Gemini (`!joseph`), Zebulun=DeepSeek (`!zebulun`), Naphtali=Mistral
  (`!naphtali`), Benjamin=Qwen (`!benjamin`), Gad=GLM (`!gad`), Levi=sim (`!levi`). **Now in code**
  (was docs-only); it's also the Slack bot's default skin (Phase 4).
- **`nightvale` — the five heads of Hiram McDaniels** (+ residents): Gold=Claude (`!gold`),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VIXAL-OS/Opus-Deipseek](https://github.com/VIXAL-OS/Opus-Deipseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
