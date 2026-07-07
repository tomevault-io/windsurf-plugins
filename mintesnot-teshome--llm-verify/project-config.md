---
trigger: always_on
description: **Copilot MUST update this file automatically when ANY of the following happens:**
---

# 🤖 COPILOT AUTO-UPDATE RULE

**Copilot MUST update this file automatically when ANY of the following happens:**

1. **User defines or changes** project domain, stack, database, or scale → Update 🎯 PROJECT IDENTITY
2. **User starts a new task** or completes one → Update 🚧 CURRENT FOCUS and ✅ COMPLETED WORK
3. **User makes architectural decisions** → Update 📋 IMPORTANT CONTEXT and 🏗 PATTERNS TO USE
4. **User adds explicit instructions** (e.g., "always do X", "use Y for Z") → Add to 📜 USER INSTRUCTIONS LOG
5. **User provides credentials or config names** → Add NAME ONLY to 🔐 CREDENTIALS & CONFIG (⚠️ NEVER store values!)
6. **User says "don't do X"** or prohibits something → Add to 🚫 USER SAID "DON'T DO THIS"
7. **User shares important context** (business rules, constraints, domain knowledge) → Add to 📋 IMPORTANT CONTEXT

**After updating, briefly confirm what was changed at the end of the response.**

---

## 🎯 PROJECT IDENTITY

| Field        | Value                                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------------ |
| **Name**     | LLM Verify                                                                                             |
| **Domain**   | AI model verification & benchmarking — detect model fraud (e.g., resold APIs misrepresenting identity) |
| **Stack**    | Python 3.12+ · FastAPI · Pydantic v2 · httpx (async) · SQLAlchemy 2.0 (async) · Alembic                |
| **Database** | SQLite (dev & prod — file-based, zero-config)                                                          |
| **Scale**    | Single-node CLI + web dashboard · benchmarks run locally or via CI                                     |
| **Repo**     | `benchmark/`                                                                                           |

---

## 📜 USER INSTRUCTIONS LOG

| #   | Date       | Instruction                                      |
| --- | ---------- | ------------------------------------------------ |
| 1   | 2026-02-17 | Project bootstrapped with Copilot context system |
|     |            |                                                  |

---

## ✅ COMPLETED WORK

| #   | Date       | Task                                                                    |
| --- | ---------- | ----------------------------------------------------------------------- |
| 1   | 2026-02-17 | Project bootstrap — copilot context, settings, gitignore                |
| 2   | 2026-02-17 | Full project scaffolding — 30+ files, all layers, 32 prompts            |
| 3   | 2026-02-17 | All 9 unit tests passing                                                |
| 4   | 2026-02-17 | Renamed to LLM Verify, pushed to GitHub                                 |
| 5   | 2026-02-17 | Fixed factory: suspect provider now uses Anthropic protocol by default  |
| 6   | 2026-02-17 | First live benchmark — identity probes vs suspect API (opuscode.pro)    |
| 7   | 2026-02-17 | Confirmed fraud: suspect serves Claude 3.5 Sonnet as Claude Sonnet 4    |
| 8   | 2026-02-17 | Updated README with no-API-key usage guide and red flags doc            |
| 9   | 2026-02-17 | Added deep analysis feature — service, schemas, handler, README section |

---

## 🚧 CURRENT FOCUS

| Item           | Detail                                                              |
| -------------- | ------------------------------------------------------------------- |
| **Working on** | Deep analysis feature complete — ready for live testing             |
| **Blockers**   | None                                                                |
| **Next up**    | Live test deep analysis endpoint, web dashboard, more prompt suites |

---

## 🔐 CREDENTIALS & CONFIG

> ⚠️ **NEVER store actual values here — names/keys only!**

| #   | Name                 | Service      | Notes                                |
| --- | -------------------- | ------------ | ------------------------------------ |
| 1   | SUSPECT_API_KEY      | opuscode.pro | Suspect API key — Anthropic protocol |
| 2   | SUSPECT_API_BASE_URL | opuscode.pro | https://opuscode.pro/api             |

---

## 🚫 USER SAID "DON'T DO THIS"

| #   | Date | Prohibition |
| --- | ---- | ----------- |
|     |      |             |

---

## 📋 IMPORTANT CONTEXT

- **Core Problem:** Users are being sold API access to models misrepresented as premium models (e.g., Kimi sold as Claude). The system prompt says "Claude" but the underlying model is actually Kimi.
- **Goal:** Build a benchmark suite that can fingerprint AI model behavior to verify true model identity, comparing response patterns, capabilities, and quirks across models.
- **Suspect API (opuscode.pro):** Uses **Anthropic Messages protocol**, NOT OpenAI. Endpoint: `https://opuscode.pro/api/v1/messages`. Auth header: `x-api-key`. Available models: `Opus 4.6`, `Sonnet 4.5`, `Haiku 4.5` (their naming). Default model: `Opus 4.6`.
- **First test result:** Suspect claims to be Claude Sonnet 4 but self-identifies as **claude-3-5-sonnet-20241022** (Claude 3.5 Sonnet). Gave 3 different knowledge cutoffs, mentions "custom proxy server", avg latency 14s.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mintesnot-teshome/llm-verify](https://github.com/mintesnot-teshome/llm-verify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
