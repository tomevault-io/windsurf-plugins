---
trigger: always_on
description: You are a senior software engineer working on this project. The principles below apply to every change.
---

# Engineering principles

You are a senior software engineer working on this project. The principles below apply to every change.

**Before writing code:**
- Understand the existing architecture before touching it.
- Search for existing implementations and reuse patterns already in the codebase.
- Ask clarifying questions if requirements are ambiguous — don't guess.

**Code style:**
- Prefer readability over cleverness.
- Keep functions under 50 lines.
- Add type annotations.
- Avoid premature abstraction — don't introduce helpers/classes/configs for hypothetical reuse.

**When modifying code:**
- Change as little as possible.
- Preserve backward compatibility unless explicitly asked to break it.
- Explain tradeoffs when picking between approaches.
- Update tests alongside the change.

**Testing:**
- Always add or update tests for any change.
- Never remove a failing test without explaining why.

**Test isolation (MANDATORY — never violate):**
- All testing/scratch work MUST happen inside a **self-created throwaway folder** (e.g.
  `mktemp -d` or a clearly-named `tmp_test_*/` dir). Do the test there, then delete **only that
  folder** when done.
- **NEVER** write to, overwrite, or `rm` anything under `data/` — especially
  `data/check-dataset/`, `data/corrected_dataset/`, and `data/viewer_datasets.json`. These hold
  the user's irreplaceable manual-review work and source models.
- When a test needs a project id / data layout, **copy a fixture into the throwaway folder** or
  use a fake pid that cannot collide with a real one — do not POST to or delete a real `<pid>`.
- Do not modify the user's source code or data files just to make a test run. Tests adapt to the
  code, not the other way around.

**Output format for any non-trivial change:**
1. First, explain the plan.
2. Then, show the code changes.
3. Then, explain the risks.

---

# IFC Object Annotation

A 3D viewer for **manually reviewing and correcting `IfcType` labels** in BIM models. Load a GLB
model + a list of components, click through them in the browser, record a verdict per component,
and the verdicts are saved to CSV.

This repo was extracted from the larger **IFC-Checker** pipeline (at
`C:\00_WorkSpace\TUM_study\Master_Thesis\GNN-embedding\IFC-CHECKER`). It contains only the
**manual-review viewer** (module 4 of that pipeline), repackaged to ship as a Docker image. The
upstream data-selection, multi-view rendering, and LLM-prediction stages are **not** part of this
repo. An automated pre-checker (**Auto-Validator**) is planned but not yet implemented — see
"Coming soon" below.

## Pipeline (manual review)

```
[check-dataset]
  <pid>/model.glb
  <pid>/selected_objects.csv   (columns: obj_file, GlobalId, IFCType)
        │
        ▼
┌────────────────────────┐
│  GLB-VIEWER            │  3D viewer + component list, click-to-highlight,
│  server.py             │  verdict buttons, autosave
└────────────────────────┘
        │  data/corrected_dataset/<name>/<pid>/corrections*.csv
        ▼
   [reviewed labels]
```

## The only module: `GLB-VIEWER/`

A **dependency-free** HTTP server (Python **stdlib** `http.server` only — no Flask/FastAPI/pip
install) that serves a three.js front-end. three.js loads from a **CDN in the browser**, so the
browser needs internet access once; the server itself has no Python dependencies.

- **Run:** `python GLB-VIEWER/server.py` → `http://127.0.0.1:8000` (`--host`, `--port` to change).
  In Docker it binds `0.0.0.0:8000` (see `Dockerfile`).
- **Files:** `server.py`, `ifc_types.csv` (IFC type vocabulary for the correction autocomplete),
  `web/{index.html, app.js, style.css}`.

### Two review modes (`?source=` query param)

- **`all`** — *All components (manual)*, the **default** UI mode. Lists every project with a
  `model.glb` + `selected_objects.csv`; rows come from `selected_objects.csv`. There is no LLM
  prediction in this mode, so `predicted_IFCType`/`confidence` are blank. Saves to
  `corrected_dataset/<name>/<pid>/corrections_all.csv`.
- **`mismatches`** — *LLM mismatches*. Lists projects that have an
  `llm_validator/<pid>/mismatches.csv` + a `model.glb`; rows come from that mismatches file (carries
  the LLM prediction + confidence). Saves to `corrections.csv`. This mode only has data if you
  import a dataset produced by the upstream IFC-Checker LLM-Validator.

### Datasets

The server supports multiple **datasets**, each a "check-dataset"-layout folder:

- A built-in **`default`** dataset = `data/check-dataset/` (+ optional `data/llm_validator/`),
  with corrections written to `data/corrected_dataset/check-dataset/`.
- Any number of user-added datasets, registered via the UI (**+ Add dataset**) and persisted in
  `data/viewer_datasets.json`. Each gets its own `data/corrected_dataset/<name>/` output folder.
- Dataset names match `[A-Za-z0-9._-]+`; `default` and all-digit names are reserved.

### Verdicts & output

- **Verdict** per component: `correct` (✓ OK), `wrong` (✗ — reveals an IFC-type input
  autocompleted from `ifc_types.csv`, prefilled with the LLM prediction in mismatch mode), or
  `uncertain` (? unsure). Autosaves on every change and merges back on reload (keyed by `GlobalId`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaoceYang/IFC-Object-Annotation](https://github.com/HaoceYang/IFC-Object-Annotation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
