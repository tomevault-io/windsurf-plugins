---
trigger: always_on
description: This file is the shared brief for anyone working in this repo, human or agentic
---

# AGENTS.md: working guide

This file is the shared brief for anyone working in this repo, human or agentic
development tool. Read it before making changes. It explains what the project is, how it
is wired, the conventions that are not obvious from the code, and the few things you must
not do. Start with the README for the deploy-and-run story, then come here for how the
pieces fit and where to change them.

## What this project is

A document processing pipeline built on Mistral models on Azure AI Foundry, with two
runtime tracks that share one `src/` codebase:

1. **Event-driven pipeline.** A file lands in blob storage, Event Grid bridges the
   `BlobCreated` event into a Storage Queue, an Azure Function
   drains the queue, and the document flows through OCR then reasoning to a JSON result.
2. **Mortgage underwriting demo with a human in the loop.** A FastAPI backend (`api.py`) with
   a separate Gradio UI (`app.py`, views `/apply`, `/review`, `/chat`) turns an application
   package into a decision. Clear cases auto-decide; borderline cases route to a reviewer.

Two models, split by role. `mistral-ocr-4` is the specialist: it turns pixels into
markdown and, in the same call, returns a `document_annotation` (type, language, summary).
`mistral-medium-3-5` is the light generalist: it extracts fields and entities, and it
underwrites against rules that live in code, not in the prompt.

## Golden rules (do not skip)

- **Python is `uv` only.** No `pip`, no `venv`, no `poetry`. Install with `uv sync`, run
  with `uv run <script>`. Add dependencies with `uv add <pkg>`, which updates
  `pyproject.toml` and `uv.lock` together.
- **Prose conventions.** No em dashes anywhere, define acronyms on first use. This
  applies to comments, commit messages, and PR descriptions.
- **Diagrams are rendered images, never ASCII art.**
- **Commit under your own name.** Keep commit messages focused on the behavior change.
- **Secrets stay out of git.** `.env`, `cases.db*`, and `.venv/` are ignored; keep it that
  way. Only the `.example` files carry placeholders. The demo sample documents under
  `inputs/` are tracked on purpose so the team can run the app and demos; generated OCR
  output (`inputs/*.result.json`, `*.md.out`) stays ignored. Do not add real customer data
  to `inputs/`; the samples there are synthetic.
- **Deploy and tear down through the scripts.** `scripts/up.sh` provisions and runs the demo
  (the azd path uses the `rg-mistral-demo` resource group); `scripts/down.sh` deletes that stack
  and purges the soft-deleted Foundry account. Do not delete resources by hand, and do not run
  `down.sh` on a stack someone else is using without asking.

## Setup

```bash
uv sync                       # create the environment from uv.lock
cp .env.example .env          # fill AZURE_AI_KEY + the storage connection string
az login                      # only needed for infra deploy or blob/queue access
```

`src/config.py` loads settings from the environment (via `.env`). It **requires**
`AZURE_AI_KEY`, `AZURE_OCR_ENDPOINT`, `AZURE_OCR_DEPLOYMENT`, `AZURE_INFERENCE_ENDPOINT`,
and `AZURE_CHAT_DEPLOYMENT`, and raises a clear error naming any that is missing. Storage
variables are optional for the CLI paths and needed only for the blob and queue flows.

## How to run each entry point

The fastest path is `scripts/up.sh`: it provisions if needed, writes `.env`, and starts both
processes (see the README). The commands below run the pieces directly for local development.

The `src/` modules import each other by **bare module name** (`from config import ...`,
`import documents as D`). So run from inside `src/`, or make sure `src/` is on
`sys.path`. The Function does the latter with a `sys.path.insert`. `uv run src/x.py`
works because the script's own directory is added to the path.

```bash
# OCR only: markdown out
uv run src/ocr.py inputs/loan_application_1003.png

# Full pipeline: OCR -> extractor -> <name>.result.json
uv run src/pipeline.py inputs/loan_application_1003.png

# Traced upload into the inbox (drives the event path once infra is deployed)
uv run src/upload.py inputs/loan_application_1003.png

# Underwriting demo: two processes, backend API + UI (start the backend first).
cd src && uv run uvicorn api:api --port 8001                              # backend service
cd src && BACKEND_URL=http://127.0.0.1:8001 uv run uvicorn app:ui --port 8000  # UI (2nd terminal)
#   Home:            http://localhost:8000/
#   Customer:        http://localhost:8000/apply
#   Reviewer:        http://localhost:8000/review
#   Document chat:   http://localhost:8000/chat
#   Backend API docs: http://localhost:8001/docs

# Azure Function locally (needs Azure Functions Core Tools)
cd function_app && cp local.settings.json.example local.settings.json && func start
```

## Architecture, module by module

### The shared OCR + reasoning core (`src/`)

| Module | Responsibility |
|---|---|
| `config.py` | Frozen `Settings` dataclass loaded from env. Builds the chat `base_url` as `<inference>/openai/v1`. |
| `ocr_client.py` | Mistral OCR 4 HTTP client. `ocr_with_annotation(data, name)` returns markdown, the annotation, page count, usage, and layout blocks. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mistralai/cookbook](https://github.com/mistralai/cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
