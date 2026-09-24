---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A self-contained, nine-phase curriculum ("AI Engineer Roadmap") for learning to build,
evaluate, and deploy LLM-powered products, plus a small set of Practice Projects that
chain multiple phases together. It is **not** a single application — it is nine
independent, runnable mini-projects (Phase 0 through Phase 8), five cross-phase practice
projects, and a two-part bonus system-design series, each in its own top-level directory
with its own dependencies, `.env`, and entry point. There is no root-level build, lint, or
test command that spans the whole repo; every command below is run from inside a specific
project directory.

Each phase pairs a theory article (`*.md`, the "why") with a working project (`*.py`, the
"how") — the article isn't just doc scaffolding, it's the source of truth for the design
decisions baked into that phase's code.

## Repository layout

Top-level directories are named `Phase_N <Title>` **with spaces** (e.g.
`Phase_4 RAG & Vector Databases`), matching the root `README.md`'s "Repository Structure"
section exactly. Every phase (0 through 8) now follows the same filenames:
`requirements.txt`, `.env.example`, `demo.py` (or `app.py`/`main.py` where a phase's own
README says so), a `README.md`, and a topic-named article (e.g. `rag_vector_databases.md`)
— there is no `ARTICLE.md` anywhere, the topic-named `.md` file *is* the article. This
convention is uniform across all 9 phases now, including Phase 3, which previously used a
`phase03_`-prefixed filename scheme; if you see a `phaseNN_`-prefixed filename referenced
anywhere, that reference is stale and should be corrected to the plain name.

Three additional top-level directories sit alongside the phases:
- `Bonus AI System Design/` — `framework_and_tradeoffs.md` and `architecture_patterns.md`,
  a two-part system-design-interview series referenced from the root README.
- `Project_1 Smart Inbox Triage/`, `Project_2 Research Copilot/`,
  `Project_3 Autonomous Content Desk/`, `Project_4 Recipe & Meal Planner/`,
  `Project_5 Travel Itinerary Planner/` — see "Practice Projects" below.

## Running a phase project

```bash
cd "Phase_N <Title>"
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env      # fill in the keys listed inside it
python demo.py            # or app.py (Phase 4 Streamlit UI) / main.py (Phase 8 FastAPI)
```

Every phase's own `README.md` is the authoritative quick-start for that phase — it names
the exact entry point and any phase-specific deviation:
- **Phase 4** and **Phase 8** also expose a Streamlit UI (`streamlit run app.py` /
  `streamlit run streamlit_app.py`).
- **Phase 7** (Fine-tuning) requires a CUDA GPU (Colab or equivalent) for
  `unsloth`/`bitsandbytes`/`torch`; its `requirements.txt` is not installable on a
  CPU-only machine, and its other `.py` files should not be imported from a non-GPU
  script for the same reason (see Project 3 below for how to work around this).
- **Phase 8** is the only phase with a Docker path: `docker-compose up --build` runs a
  FastAPI backend (`main.py`) + Streamlit frontend (`streamlit_app.py`) together, wired to
  Langfuse observability (`observability.py`) and a RAGAS evaluation gate.

## Practice Projects — cross-phase, not standalone

`Project_1 Smart Inbox Triage/`, `Project_2 Research Copilot/`,
`Project_3 Autonomous Content Desk/`, `Project_4 Recipe & Meal Planner/`, and
`Project_5 Travel Itinerary Planner/` each combine 2-3 adjacent phases into one project.
Critically, **they import the relevant phase's actual code directly** rather than
duplicating logic — each project's core module (`triage.py`, `copilot_graph.py`,
`crew.py`, `meal_planner.py`, `itinerary_planner.py`) does this at the top of the file:

```python
_REPO_ROOT = Path(__file__).resolve().parent.parent
sys.path.insert(0, str(_REPO_ROOT / "Phase_N <Title>"))
from some_module import SomeClass   # imported straight from that phase's folder
```

This means:
- **Renaming or restructuring a file inside a phase folder can silently break a Practice
  Project's import.** Before renaming anything inside `Phase_1`, `Phase_2`, `Phase_3`,
  `Phase_4`, `Phase_5`, or `Phase_6`, grep the `Project_*` folders for imports of that
  phase's modules.
- **Project 3 deliberately does NOT import anything from Phase 7.** Phase 7's `train.py`
  and `inference_api.py` depend on CUDA-only packages (`unsloth`, `bitsandbytes`) that
  won't import on a non-GPU machine. Instead, `Project_3/crew.py` calls Phase 7's fine-tuned
  model over plain HTTP (`FINETUNED_MODEL_URL`), the same way Phase 7's own
  `demo_client.py` does, and degrades gracefully if that URL is unset or unreachable. Keep
  this pattern if you touch that code — do not add a direct import of Phase 7 modules.
- Run `python -m py_compile <file>.py` after editing any Practice Project module as a
  cheap sanity check; a full run additionally needs the target phase(s)' dependencies
  installed and real API keys, since these projects make live LLM calls with no mocking.

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shikharkumar13/AI-Engineering-Bootcamp](https://github.com/shikharkumar13/AI-Engineering-Bootcamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
