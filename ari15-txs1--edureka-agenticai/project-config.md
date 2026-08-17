---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This is a learning/reference repository for an "Agentic AI" course, not a single deployable application. Each top-level numbered directory is a standalone playground for one framework or topic, containing many small, independent scripts (mostly one script = one demo/concept). There is no shared build system, no test suite, and no linter configuration — treat each script as a self-contained example.

## Setup & running scripts

- Root `requirements.txt` covers dependencies for most numbered directories (OpenAI, LangGraph/LangChain, CrewAI, AutoGen, MCP, Bedrock, RAG/vector-store libs, explainability, etc.), pinned to specific versions verified to both resolve (`pip install --dry-run`) and actually import correctly in a real venv (`uv venv` + `uv pip install -r requirements.txt`). Install with `pip install -r requirements.txt` (or `uv pip install -r requirements.txt`) into a venv.
- The LangChain/LangGraph/CrewAI packages are deliberately pinned to their pre-1.0 lines (langchain-core 0.3.x, langgraph 0.6.x, crewai 0.203.x) because the repo's scripts use 0.x-era APIs that break under the coordinated "1.0" releases those projects shipped. `chromadb` and `numpy` are held below their absolute latest for the same reason — see the inline comments in `requirements.txt`.
- **`dspy` is intentionally excluded** from `requirements.txt`: `crewai` hard-pins `json-repair==0.25.2` while every `dspy` release requires `json-repair>=0.30.0`, so the two can never coexist in one environment. If you need to run `9_general/dspy/dspy_1.py` or `dspy_2.py`, create a separate venv and `pip install dspy` there.
- **Known landmine: `crewai` breaks `shap` if imported first in the same process.** `crewai` monkey-patches `warnings.warn` to suppress pydantic deprecation noise, and that patch doesn't forward the `skip_file_prefixes` kwarg matplotlib uses internally — so `import shap` after `import crewai` raises `TypeError: filtered_warn() got an unexpected keyword argument 'skip_file_prefixes'`. Harmless as long as `4_crewai/` and `9_general/explainability/` scripts stay in separate processes (which they are today); if you ever combine crewai and shap in one script, import `shap` first.
- phidata's import name is `phi`, not `phidata` (`import phi`, `from phi import ...`) — don't be misled by the package name when checking whether it's installed.
- Some subprojects ship their own isolated `venv/` (e.g. `12_project/12_1_openai_agents_diet_agent/venv`) — check for a local venv before assuming the root install applies.
- Root `.env` holds placeholder keys for every provider/tool used anywhere in the repo (OpenAI, Anthropic, Google, Groq, Cohere, Pinecone, Qdrant, Tavily, Exa, SerpAPI, MailerSend, ntfy, etc.). Nearly every script calls `load_dotenv(override=True)` near the top, expecting `.env` in the repo root (Google ADK agent folders instead keep their own local `.env`).
- There is no single entrypoint. Scripts are run individually, e.g. `python 3_langgraph/3_1_langgraph_basic.py`.

## Known gotcha: hardcoded absolute paths

Many scripts (MCP server paths passed to subprocesses, Chroma persistent-client directories, image/data file paths, etc.) hardcode absolute Windows paths under the repo's *old* folder name, `C:\code\agenticai\...` (missing the `-claude` suffix this repo now has). This affects ~50 files spread across `1_openai_chat_requests`, `2_openai_agents`, `3_langgraph`, `4_crewai`, `6_mcp`, `8_bedrock`, `9_general`, and `12_project` — including the Flask demos, which intentionally kept this stale path style to match the sibling files they were ported from. If a script fails with a file-not-found error, check for one of these stale paths first and update it to the current repo root (or better, make it relative via `os.path`).

## Directory map

- `0_slides` — course slide deck (`Agentic AI.pptx`); not code.
- `demo` — a single standalone OpenAI Agents SDK warm-up script (`our_first_agent.py`), separate from the numbered course directories.
- `1_openai_chat_requests` — raw OpenAI Chat Completions and Responses API examples, plus Gemini/Ollama equivalents (streaming, images, PDFs, structured output via Pydantic, a basic chatbot).
- `2_openai_agents` — OpenAI Agents SDK: classic agent patterns (tool use, plan-and-execute, ReAct, reflection, ReWOO, multi-agent), RAG variants (hardcoded/semantic/OpenAI vector store/FAQ db), guardrails, sync/async agents, agent-as-tool, short-term memory, and Flask front ends for an insurance/FAQ demo.
- `3_langgraph` — LangGraph: LCEL basics, memory strategies (ephemeral, `MemorySaver`, SQLite checkpointer), a code-review agent, guardrails, and a banking chatbot built up across ChromaDB/FAISS/SERP/email-notification variants with Flask front ends.
- `4_crewai` — CrewAI: multi-agent crews for document generation, log analysis, cloud billing, stock analysis, and customer service (the latter two with Flask front ends), including with/without-memory comparisons.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ari15-txs1/Edureka_AgenticAI](https://github.com/ari15-txs1/Edureka_AgenticAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
