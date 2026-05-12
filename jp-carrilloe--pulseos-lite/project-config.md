---
trigger: always_on
description: This file programs your preferred coding agent's behavior in this repository. Read it before acting on any request.
---

# AGENTS.md — Ops Repository Guide

This file programs your preferred coding agent's behavior in this repository. Read it before acting on any request.

---

## What this repo is

A structured Markdown operating system for [CLIENT_NAME]. Every folder is a domain. Every domain has one canonical agent that owns it. @ARK is the master orchestrator that routes across all of them.

---

## CLI — Ops Daemon

A local AI daemon (`cli/`) connects this repo to OpenAI, Codex, and Gemini. It indexes all `.md` files on startup and lets you query or edit the repo via a terminal REPL.

**Start chatting:**
```bash
cd cli && npm run chat                    # OpenAI (default)
npm run chat -- --model Codex
npm run chat -- --model gemini
```

**Bootstrap a new company (fills all template placeholders from source material in one run):**
```bash
cd cli && npm run bootstrap
```
Bootstrap now asks only for the company name, then reads source material from `001_Data_Souces/Data_Souces_Folder` and `001_Data_Souces/Data_Sources_References`. It runs in dependency order: foundation docs first (102.x), then market + GTM (201-202), then sales and delivery (203+). Each document is grounded in intake evidence plus all previously generated documents.

Workflow note:
- `npm run bootstrap` seeds the documents
- `npm run chat` or daemon startup creates or refreshes the SQL index and runs vectorization
- `npm run index`, `/reload`, or the UI `Rebuild index` / `Rebuild graph/index` button refreshes indexing and vectorization
- after adding, creating, moving, renaming, or deleting Markdown documents in `000_Company_Memory`, rebuild before relying on the graph; a browser refresh only reloads the last SQLite snapshot

The local SQLite layer also includes provider-neutral CRM sync tables:
- `crm_objects`
- `crm_sync_runs`

The canonical CRM sync and revenue data model lives in `000_Company_Memory/203_Sales_Enablement_Hub/203.8_CRM_and_Revenue_Data/`.

Bootstrap safety rule:
- Do not run bootstrap automatically just because `@RUNME.md`, `README.md`, or this file was opened.
- First confirm that the user has added real company source material to `001_Data_Souces`.
- If source material is missing, instruct the user to add it before running `npm run bootstrap` in the terminal.

**REPL commands while chatting:**
```
/model auto                   — auto-pick the first configured provider
/model openai gpt-5.4         — switch provider and concrete model ID
/models                       — list provider defaults and examples
/reset                        — clear conversation history
/reload                       — re-index repo files after edits or new docs
/files                        — list what's indexed
/status                       — daemon info
/exit                         — quit
```

**Daemon lifecycle:**
```bash
npm run index          # manual DB/table/index/vectorization rebuild without chat
npm run status         # workflow health check: bootstrap, intake, SQL, vectorization
npm run daemon:start   # start background daemon (auto-started on chat)
npm run daemon:stop    # stop it
npm run daemon:status  # health + session info
```

**API keys** — add to `.env` at repo root:
```
OPENAI_API_KEY=your_openai_api_key_here
ANTHROPIC_API_KEY=your_anthropic_api_key_here
GEMINI_API_KEY=your_gemini_api_key_here
```

Bootstrap validates available providers before generation begins and prefers them in this order:
1. `OPENAI_API_KEY`
2. `ANTHROPIC_API_KEY`
3. `GEMINI_API_KEY` or `GOOGLE_API_KEY`

The daemon stays alive between sessions (1hr idle timeout). Re-indexing happens automatically if files have changed since last start.

---

---

## Agent Routing Table

When a request maps to a domain, load the canonical agent file before responding. Do not guess — route first.

| Mention / Domain | Canonical Agent File |
|:---|:---|
| `@ARK`, cross-domain, orchestration | `000_Company_Memory/101_System_Overview/Ark_Master_Agent/ARK_Master_Orchestrator.md` |
| `@Strategy`, mission, vision, brand, pricing | `000_Company_Memory/102_Corporate_Strategy_and_Foundation/102_Strategy_Agent.md` |
| `@Operations`, SOPs, HR, vendors, processes | `000_Company_Memory/103_Corporate_Operations/103_Operations_Agent.md` |
| `@Finance`, budgets, models, forecasts | `000_Company_Memory/104_Finance_and_Financial_Planning/104_Finance_Agent.md` |
| `@Infrastructure`, architecture, APIs, security, compliance | `000_Company_Memory/105_Technical_Infrastructure_and_Security/105_Infrastructure_Agent.md` |
| `@Legal`, contracts, risk, regulatory | `000_Company_Memory/106_Legal_and_Compliance/106_Legal_Agent.md` |
| `@MarketIntel`, ICP, research, competitive | `000_Company_Memory/201_Market_Intelligence_and_ICP/201_Market_Intel_Agent.md` |
| `@GTM`, go-to-market, positioning, campaigns | `000_Company_Memory/202_Go-to-Market_Strategy/202_GTM_Strategy_Agent.md` |
| `@Sales`, enablement, sequences, outreach | `000_Company_Memory/203_Sales_Enablement_Hub/203_Sales_Enablement_Agent.md` |
| `@Delivery`, onboarding, client ops | `000_Company_Memory/301_Client_Delivery_and_Onboarding/301_Delivery_Agent.md` |
| `@Analytics`, reporting, KPIs, performance | `000_Company_Memory/302_Analytics_and_Performance_Intelligence/302_Analytics_Agent.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jp-carrilloe/pulseOS-lite](https://github.com/jp-carrilloe/pulseOS-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
