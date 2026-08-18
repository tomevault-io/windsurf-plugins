---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Always read `plan.md` at session start to know current phase and next step — it's the living
record of progress across sessions, so check it before assuming what's done vs. still open.

## Commands

Run all commands from this directory (`rag_agents_cover_letter/`) — `pytest.ini` sets `pythonpath = .` so `src` resolves as a package from here.

```bash
# Activate the existing venv (or create one: python -m venv .venv)
source .venv/bin/activate
pip install -r requirements.txt

# Run all tests
pytest

# Run a single test
pytest tests/test_ingest.py::test_load_pdf_documents_reads_pdf_with_cover_letter_metadata -v

# Run one test file
pytest tests/test_retrieve.py -v

# Full ingest pipeline (real OpenAI calls: gpt-4o-mini metadata extraction per PDF + embeddings — costs money, requires OPENAI_API_KEY in .env)
python -m src.ingest

# CLI query against the persisted vector store
python -m src.retrieve "some query"

# Wipe the vector store to force a clean re-ingest
rm -rf chroma_db

# Run the eval harness (real gpt-4o-mini calls: full agent graph + 2 judge calls per example — costs money, appends to eval/results.csv)
python -m eval.run_eval
```

Notebook (`notebooks/explore.ipynb`) outputs are stripped via `nbstripout` (declared in `.gitattributes`). After cloning, run once: `pip install nbstripout && nbstripout --install`.

## Architecture

Three-stage pipeline, one module each, composed either manually or via the `agent/` LangGraph graph below:

**`src/ingest.py`** — load → chunk → embed → persist.
- PDFs live under `data/cover letters/`, `data/projects/`, `data/cv/`; each directory has a thin wrapper (`load_pdf_documents`, `load_project_documents`, `load_cv_documents`) around the shared `_load_pdf_documents_from(directory)`.
- For every PDF, `extract_metadata_llm(text, source)` calls `gpt-4o-mini` in JSON mode to derive `type` (`cv`/`project`/`cover_letter`), `company`, and `skills` (comma-separated tools only). This dict becomes the `Document.metadata`, which `RecursiveCharacterTextSplitter.split_documents()` then propagates onto every chunk — so metadata is per-source-document, not per-chunk-derived.
- `ingest()` chunks with `RecursiveCharacterTextSplitter` (300/50, paragraph → line → sentence separators), embeds with `OpenAIEmbeddings(text-embedding-3-small)`, and persists to a Chroma collection (`cv_documents`) at `chroma_db/`. It calls `.delete_collection()` first — re-running `ingest()` replaces the collection rather than appending duplicate chunks.
- `load_documents()`, `chunk_documents()`, and `infer_type()` are a legacy token-count-based path over `data/*.txt` with filename-keyword type inference. They're no longer wired into `ingest()` (see the commented-out line) but are still covered by tests — don't assume they're dead code without checking call sites first.

**`src/retrieve.py`** — query the persisted collection.
- `get_retriever(k, doc_type=None)` builds a Chroma similarity retriever; passing `doc_type` adds a metadata `filter={"type": doc_type}`.
- `retrieve(query, k)` is unfiltered top-k; `retrieve_by_type(query, doc_type, k)` filters by the `type` field. There is no equivalent filter for `company` yet, and any chunks embedded before `extract_metadata_llm` existed won't have a `company` field at all.

**`src/generator.py`** — `generate(context, job_description, gap_analysis="", company_research="")` sends retrieved context + a job description + any known gaps + company research through a fixed prompt to `gpt-4o-mini` (temperature 0.3) and returns one cover-letter paragraph. The prompt explicitly forbids inventing facts not present in the supplied context, instructs the model to address noted gaps honestly (leaning on adjacent experience) rather than ignore or overclaim them, and to use company research naturally rather than forcing it in. Both extra params default to `""`, which become `"None noted."`/`"None."` in the prompt respectively.

**`agent/`** — LangGraph agent composing the pipeline above into a graph (`plan.md` Phase 2, now complete).
- `agent/state.py` defines `AgentState` (TypedDict): `job_description` + optional `company` in, `retrieved_chunks`/`company_research`/`gap_analysis`/`cover_letter` populated by the graph below.
- `agent/nodes/retrieve.py` and `agent/nodes/generate.py` wrap `src.retrieve.retrieve`/`src.generator.generate` directly — no logic duplicated. `agent/nodes/gap_analysis.py` mirrors `src/generator.py`'s own `ChatPromptTemplate` + `ChatOpenAI` shape (`temperature=0`, factual comparison) to flag what's missing between retrieved chunks and the job description. `agent/nodes/company_research.py` wraps a module-level `TavilyClient()` (reads `TAVILY_API_KEY` from env — eager construction, no guard, same convention as `ingest.py`'s `_openai_client`) and calls `.search(..., include_answer=True)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danatok/rag_agents_cover_letter](https://github.com/danatok/rag_agents_cover_letter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
