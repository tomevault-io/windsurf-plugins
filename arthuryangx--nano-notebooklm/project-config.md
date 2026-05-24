---
trigger: always_on
description: This file is for **AI coding assistants** (Claude Code, Cursor, Codex,
---

# Using nano-NOTEBOOKLM with an AI coding assistant

This file is for **AI coding assistants** (Claude Code, Cursor, Codex,
Copilot, …) that you ask to extend, debug, or operate this codebase.
Hand it the path `CLAUDE.md` and it will know enough to make safe,
targeted changes.

Humans: see [`README.md`](README.md) for install + usage.

---

## What this project is

A self-hosted study assistant that ingests course documents (PDF / PPTX
/ DOCX / Markdown) and provides chat, structured notes, quizzes, an
exam-prep loop, and an editable knowledge graph — all backed by a
provider-agnostic LLM router.

- Single-process FastAPI + React 18 (CDN, no build).
- No DB, no auth, no multi-tenant — everything lives in `./artifacts/`.
- Default LLM backend is OpenAI-compatible; Anthropic Claude and any
  local OpenAI-compatible server (Ollama / vLLM / LM Studio / llama.cpp)
  are first-class siblings.

---

## Repo layout — where to make changes

```
api/server.py                    FastAPI routes + middleware + Pydantic models.
                                 ~5300 lines; one file deliberately. Search by route.

frontend/                        React 18, no bundler. Edit a .jsx and reload.
  app.jsx                        Top-level shell, course switching, topbar chips.
  assistant.jsx                  Chat sidebar.
  reader.jsx                     PDF/PPTX viewer + citation jump-to-page.
  notes.jsx                      LaTeX notes editor (CodeMirror) + tectonic PDF.
  mindmap.jsx                    d3-force KG with edit ops.
  exam-prep.jsx                  Self-evolving quiz.
  quiz.jsx                       Practice quiz (one-shot, non-bank).
  library.jsx                    Course library sidebar + course picker modal.
  processing.jsx                 Upload progress overlay (consumes NDJSON stage events
                                 from /api/upload + the ETA estimator from server.py).
  settings.jsx                   Providers matrix, embedding-preset radios, status badges.
  tweaks-panel.jsx               Per-course generation tweaks (note/quiz/exam knobs).
  i18n.js                        Central STRINGS table (zh + en). All user-facing copy
                                 goes here; components call `t("key", {placeholders})`.
                                 New strings: add to the dict + reference via t(); never
                                 inline a literal in JSX. Missing key falls back to the key.
  api.js                         Fetch wrappers; one place to add a new endpoint client-side.
  study-state.js                 Shared client state (active course / file / chat session).
  markdown.js                    Markdown renderer (assistant answers, notes preview).
  latex-to-html.js               LaTeX → HTML sanitizer for KaTeX rendering.
  styles.css                     All CSS lives here.

nano_notebooklm/
  ai/
    base.py                      LLMBackend ABC (carries name + kind attrs) + TruncationSignal.
    openai_backend.py            Generic OpenAI-compatible client (OpenAI, DeepSeek, Moonshot,
                                 Zhipu, MiniMax, Groq, Together, Gemini compat endpoint, …).
                                 Detects base_url to handle codex-proxy and DeepSeek quirks.
                                 Accepts http_timeout kwarg so /api/providers/{id}/test can
                                 cap the underlying httpx client.
    claude_backend.py            Anthropic native API (same http_timeout kwarg).
    local_backend.py             Legacy thin subclass; new code paths construct OpenAIBackend
                                 directly via _build_backend (kept for backward import compat).
    router.py                    ModelRouter — reads artifacts/providers.json, dispatches by
                                 task_type or explicit override, handles retries + fallback.
                                 Methods: reload() to swap backends without restart,
                                 get_openai_compat() for endpoints that need a chat-completions
                                 backend (agent, explain-node), diagnose_row() for UI badges.
    prompt_templates.py          All system prompts and language bindings.

  ingest/                        File → chunk pipeline (PDF/PPTX/DOCX/MD).
  kb/                            FAISS + BM25 + RRF + graph_search (KG-driven retriever).
  kg/                            Two-stage KG extraction (topics → leaf concepts).
  skills/                        Independent business logic, each exposes execute(params).
                                 Filenames are inconsistent (historical) — don't rename
                                 for "consistency"; tests grep by exact filename.
    qa_skill.py                  /api/chat — intent routing, graphrag, RAG, translation,
                                 cross-course, general; the largest skill.
    notes_full_course.py         Per-file LaTeX generation with incremental cache.
    note_generator.py            Legacy single-shot note skill (kept for /api/notes).
    quiz_generator.py            Practice quiz.
    exam_prep.py                 Topic plan → seed questions → quiz draw → submit + variant.
    exam_analyzer.py             One-shot exam-pattern analysis (older surface).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArthurYangX/nano-NotebookLM](https://github.com/ArthurYangX/nano-NotebookLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
