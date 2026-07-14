---
trigger: always_on
description: This file is for GPT-family or Codex-family coding agents working on this repository. The longer canonical guide for architecture, feature roadmap, and known issues is `CLAUDE.md`. Read it for context before doing non-trivial work.
---

# AGENTS.md — GPT / Codex Contributor Guide

This file is for GPT-family or Codex-family coding agents working on this repository. The longer canonical guide for architecture, feature roadmap, and known issues is `CLAUDE.md`. Read it for context before doing non-trivial work.

## Project Summary

**Little Lantern** is a local-first chat frontend for non-coders to use multiple LLM API providers through one interface. Vanilla HTML/CSS/JS, no build tools, no npm. It runs at `http://127.0.0.1:3000` via `start.py` (or `start.bat` on Windows).

This is **not** an assistant framework, not a server daemon, not a background process. It is a single-page web app the user runs on their own machine.

## Build Status (updated 2026-07-11) — check the code before trusting any checkbox

The app itself is feature-complete. **Built and working:** all six API providers (OpenAI, Claude, Nous, OpenRouter, Gemini, Mistral) with proper `system` + `messages` formatting; structured tools (calculator, web_search, url_fetch, file_read, file_search, file_write, image_generate, memory_add) on tool-capable providers; per-model sampler/effort handling; finalised model dropdowns; the GM Notes hidden channel; a cumulative token odometer; and prompt caching where supported. **Newer additions (all in `app.js`):** the auto-memory engine (`memory_add` + a per-companion Desk Notes file read fresh each turn), an idle **heartbeat** (off by default), full **JSON state export/import** (API keys excluded), top-bar token spend, and a ❓ Help modal. A private beta went out 2026-06-15; beta-feedback fixes landed 2026-06-22.

**Built 2026-06-28 (`5ed04b8`):** a per-companion **Voice & Interaction Examples** upload (an optional sample-exchange file, used when moving a companion in from another app) — it replaced the redundant *manual* Desk Notes upload; voice examples and the auto Desk Notes file now load together in the stable bundle. Same pass: default temperature ships at 1; UI example filenames tidied to `companionname-context.md`.

**Built 2026-06-30:** local server hardening and memory reliability fixes. `start.py` binds to `127.0.0.1` only, CORS is restricted to loopback origins, and incoming filenames are URL-decoded before basename/safety checks. Test Connection now sends a real small validation request for every supported provider, including Gemini and Mistral, and `API_REFERENCE.md` was updated as the source of truth. Auto-memory now has code-side near-duplicate protection: `memory_add` skips/merges duplicate memories, recalled memory is deduped before prompt injection, heartbeat tells the model to save only genuinely new durable facts, and a per-chat **Memory operation ledger** carries actual `memory_add` results into later turns. Live Lumen testing passed same-turn tool receipt visibility, already-exists/trigger-merge behaviour, and next-turn ledger visibility.

**Done 2026-07-01:** `claude-sonnet-5` added to the Claude dropdown (adaptive/effort handling — no samplers). The full guide set was audited against the live code and committed. **Lumen the Diagnostic Octopus ships** as an optional example companion + diagnostic helper (`system-prompts/Lumen_System_Prompt.txt`, backup in `Little_Lantern_Guides/Lumen_Diagnostic_Octopus_Backup/`; optional/deletable, noted in README and START_HERE). UI display text updated: the Working Directory is labelled as the companion's sandbox folder, and the companion About field no longer mentions rules — behavioural rules belong in the System Prompt, not the card. Note: the About You (persona) **Name** field is UI-only; only the persona Description is sent to the model.

**Done 2026-07-01 (evening) / 2026-07-02:** **Heartbeat receipts** — each beat records which tools it actually called (`currentChat.heartbeatReceipts`, injected as `### Heartbeat receipts:` alongside the memory ledger); live-tested. **"Current Context" renamed to "Desk Notes"** in all display text, prompt-injection headers, and docs (it collided with SillyTavern's "recent context"). **Internal identifiers deliberately unchanged** (`currentContextWorkingFile`, `getCurrentContextContent()`, `CURRENT_CONTEXT_MAX_CHARS`, DOM id `characterCurrentContextWorking`) — display-only rename; do NOT rename the internals.

**Done 2026-07-10:** GPT-5.6 Sol/Terra/Luna added to OpenAI with reasoning detection and a 5.6-only `max` effort option; Grok 4.5 added through OpenRouter with a Low/Medium/High reasoning selector (High default). Every public provider now has a persistent custom model-ID slot so same-provider model names can be used without another dropdown edit (request-shape changes still require code). The Memory Book editor's **+ Add Entry** button moved beneath the entry list so long books do not require scrolling back to the top to add another memory. Gemini 3.5 Pro is documented as coming soon but is not hardcoded until Google publishes its exact Gemini API ID.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MadScientist-Labs/little-lantern](https://github.com/MadScientist-Labs/little-lantern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
