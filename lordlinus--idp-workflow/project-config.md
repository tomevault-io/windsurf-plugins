---
trigger: always_on
description: pip install -r requirements.txt
---

# Copilot Instructions — IDP Workflow

## Build & Run

### Backend (Azure Functions, Python 3.13+)

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally (requires Azure Functions Core Tools v4)
func start
```

### Frontend (Next.js, in `frontend/`)

```bash
cd frontend
npm install
npm run dev          # Dev server on :3000
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # TypeScript strict check
```

There is no Python test suite or linter configured for the backend. API testing is done via `tests/demo.http` (VS Code REST Client) or curl.

## Architecture

This is an **Azure Durable Functions** orchestration that processes documents through a 6-step pipeline with real-time UI updates via SignalR.

### Pipeline Flow

```
HTTP POST /api/idp/start
  → Orchestrator (idp_workflow/orchestration/orchestration.py)
    → Step 1: PDF Extraction (Azure Document Intelligence → Markdown)
    → Step 2: Classification (DSPy ChainOfThought)
    → Step 3: Data Extraction (Azure CU + DSPy, run concurrently)
    → Step 4: Comparison (Azure vs DSPy field-by-field)
    → Step 5: Human Review (HITL gate — waits for external event or timeout)
    → Step 6: AI Reasoning Agent (validation, summary, recommendations)
  → Final result returned
```

Each step broadcasts `stepStarted` → `stepCompleted`/`stepFailed` events via SignalR so the frontend can update in real-time.

### Backend Layers

- **`function_app.py`** — Entry point. Registers activities, orchestration, HTTP endpoints, and SignalR via modular `register_*()` functions.
- **`idp_workflow/constants.py`** — Step registry. `STEPS` tuple of `StepInfo` namedtuples is the single source of truth for step IDs, display names, numbers, and activity names. `STEP_META` dict derives from it. Individual constants (`STEP1_PDF_EXTRACTION`, etc.) kept for backward compat.
- **`idp_workflow/errors.py`** — Typed error hierarchy. `IDPError` base → `ExtractionError`, `ClassificationError`, `ComparisonError`, `ReasoningError`, `ConfigurationError`. All carry `request_id` and `step_name`.
- **`idp_workflow/orchestration/`** — Durable orchestrator. Uses `_execute_step()` helper for standardized broadcast→activity→broadcast flow, `yield context.task_all(...)` for parallel execution, and `wait_for_external_event()` with timer race for the HITL gate.
- **`idp_workflow/activities/activities.py`** — Activity functions (one per step). Each uses `ActivityContext` for request_id extraction, timing, and structured logging.
- **`idp_workflow/activities/utils.py`** — `ActivityContext` helper class. Standardizes request_id extraction, elapsed time measurement, structured logging (`log_start`/`log_complete`/`log_error`), and result formatting.
- **`idp_workflow/steps/`** — Step logic classes (`PDFMarkdownExtractor`, `DocumentClassifier`, `AzureExtractor`, `DSPyExtractor`, etc.). Each exposes an `async` method returning `(PydanticModel, step_output_dict)`.
- **`idp_workflow/api/endpoints.py`** — HTTP + SignalR endpoints. Uses `@app.durable_client_input()` for orchestration control.
- **`idp_workflow/domains/`** — Domain-specific configs (e.g., `insurance_claims/`, `home_loan/`). Each domain has `config.json`, `classification_categories.json`, `extraction_schema.json`, and optional `validation_rules.json`. Loaded via `domain_loader.py` with LRU caching.
- **`idp_workflow/tools/`** — Shared AI utilities: `AzureContentUnderstandingClient` (REST wrapper), DSPy helpers for dynamic Pydantic model generation from extraction schemas.

### Frontend Layers

- **State**: Four Zustand stores with Immer middleware — `workflowStore` (steps, HITL state), `eventsStore` (SignalR event log), `reasoningStore` (streaming chunks), `uiStore` (connection, toasts).
- **Step config**: `lib/stepConfig.ts` — single source of truth for all step UI metadata (display names, order, numbers, icons, descriptions, pipeline layout). Exports `STEP_CONFIGS`, `STEP_ORDER`, `STEP_DISPLAY_NAMES`, `STEP_INFO`, `PIPELINE_ROWS`.
- **Shared utilities**: `lib/formatting.ts` — `formatFieldValue()` and `getConfidenceColor()` extracted from components.
- **API**: `apiClient.ts` (Axios singleton) and `signalrClient.ts` (auto-reconnect with exponential backoff).
- **Components**: `FileUploadArea`, `WorkflowDiagram` (Reaflow visualization), `HITLReviewPanel`, `ReasoningPanel`, `detail/` (split `DetailPanel` → container + 8 focused sub-components: `StepOutputRenderer`, `StepOutputView`, `ReasoningOutput`, `ValidationRulesPanel`, `ExtractionSchemaView`, `CompletionDashboard`, `DefaultView`, `ValueDisplay`).
- **Data fetching**: React Query hooks (`useUploadPDF`, `useStartWorkflow`, `useDemoDocument`).

## Key Conventions

### Activity function pattern

Every activity uses `ActivityContext` from `idp_workflow/activities/utils.py`:

```python
@app.activity_trigger(input_name="request_name")
async def activity_step_XX_name(request_dict: dict) -> dict:
    ctx = ActivityContext(request_dict, "step_XX_name")
    ctx.log_start()
    try:
        # Instantiate step class, call async method
        result, step_output = await executor.method(...)
        ctx.log_complete()
        return ctx.result(extraction_result=result.model_dump(), step_output=step_output)
    except Exception as e:
        ctx.log_error(e)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lordlinus/idp-workflow](https://github.com/lordlinus/idp-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
