---
trigger: always_on
description: **All commands must use `uv run` prefix** — bare `pytest`/`python` uses the wrong environment.
---

# CLAUDE.md

## Gate: `uv run poe lint`

**All commands must use `uv run` prefix** — bare `pytest`/`python` uses the wrong environment.

## Project Context
- **v0.6.0**: breaking release — API changes allowed.
- **Breaking commits**: use `!` in the type (e.g. `feat(data)!:`, `refactor!:`) and include a `BREAKING:` line in the body describing the API change.
- **Task tracking**: Linear (project `nltools`, team `Ejolly`).

## Skills

Use the vendored project skills (in `.claude/skills/`) for the domains they cover — don't work from memory:
- **`nilearn`** — before writing, reviewing, or debugging any nilearn code (GLM, masking, plotting, decoding, datasets, connectivity). This codebase builds directly on nilearn; the skill carries current signatures and patterns.
- **`marimo-notebook` / `marimo-pair`** — when authoring or editing the marimo `.py` tutorials under `docs/tutorials/`.

## Architecture: Functional Core, Imperative Shell

Classes are **facades and glue** — all real logic lives in pure functions.

- **Shell** (imperative): `nltools/data/` — `BrainData`, `Adjacency`, `DesignMatrix`, `BrainCollection`. Each is a facade over submodules (io, modeling, plotting, etc.)
- **Core** (functional): `stats`, `utils`, `cross_validation`, `mask`, `algorithms/` (`alignment` [SRM/hyperalignment/LocalAlignment], `inference`, `ridge`)

**Design rules:**
- Pure functions first. Classes compose and delegate to them, never the reverse.
- Use frozen dataclasses for immutable state containers. Prefer modern Python (type hints, `@dataclass(frozen=True)`, `|` unions, etc.).
- Don't repeat logic — extract shared helpers as functions where most useful and import them. Prefer a single source of truth over duplicated code.
- **No underscore-prefixed module names** (e.g. `validation.py` not `_validation.py`). Leading underscores are fine for internal functions/methods, just not filenames.

**Internals reference** (design docs for the subsystems below — read the relevant one before changing that subsystem; keep it in sync when behavior changes):
- `docs/development/execution-model.md` — `BrainCollection` parallel execution: path-backed caching, the `cache=` knob, HDF5 fit bundles, `_ItemTask`/`_DesignContext` pickling, parallel write safety. (Replaces the old `data/collection/SPEC.md`.)
- `docs/development/ridge-internals.md` — the six ridge tricks + the `class Backend` abstraction (`parallel=`/hyphenated names/MPS). (Replaces `algorithms/ridge/{DESIGN,README}.md`.)
- `docs/development/inference-internals.md` — permutation/bootstrap algorithms, deterministic cross-backend RNG, Phipson-Smyth p-values, numerical stability. (Replaces `algorithms/inference/DESIGN.md`.)
- `docs/development/index.md` — the architecture overview + canonical kwarg vocabulary (human-facing entry point; the interactive Design Tour at `docs/public/design-tour.html` links into these).

## API Conventions (v0.6.0)

Canonical kwarg names across the four data-class facades:

| Concept | Canonical kwarg | Notes |
|---|---|---|
| Algorithm/variant choice | `method` | not `algorithm`, `scheme`, `kind`, `estimator`, `icc_type`, `extract_type`, `perm_type`, `mode` |
| Spatial scale | `spatial_scale` | values: `'whole_brain' \| 'roi' \| 'searchlight'` (a given method may support a subset and raise `NotImplementedError` for the rest — e.g. `BrainData.align` has no `searchlight`; `BrainCollection.align` / `LocalAlignment` are local-only, no `whole_brain`). Used by `BrainData.predict` / `.distance` / `.align` / `.mean`/`.std`/`.median`, `BrainCollection.predict` / `.align`, and `LocalAlignment` (with companion `roi_mask=`). Distinct from `method=` (algorithm choice). Vocabulary follows Jolly & Chang, 2021, *SCAN*. |
| Distance/similarity metric | `metric` | kept separate from `method` |
| Parallel execution | `n_jobs: int = -1` | not `parallel=` (stats-layer internals still use `parallel=` but facades translate) |
| GPU/CPU selection | `device: str = "cpu"` | BrainCollection only; separate from `n_jobs` |
| Progress indicator | `progress_bar: bool = False` | not `show_progress`, `verbose` (`verbose` reserved for log-level only) |
| Threshold pair | `lower`, `upper`, `binarize` | plus convenience `threshold: float` where bidirectional |
| Permutation count | `n_permute` | not `n_perm`, `n_iter` |
| Bootstrap sample count | `n_samples` | semantically distinct from `n_permute` |
| Diagonal flag | `include_diag: bool` | not `ignore_diagonal` |
| Radius (mm) | `radius_mm: float` | units in the name |

**Canonical trailing kwarg order** (when any apply):
`..., domain_kwargs, return_flags, n_jobs=-1, random_state=None, progress_bar=False`

**`**kwargs` rule**: permitted **only** when forwarding to an external third-party API (sklearn estimator, matplotlib, nilearn, nibabel, seaborn, pandas). Internal delegation between nltools modules must use explicit signatures.

**Keyword-only `*` marker**: required in `__init__` after the primary data arg, and in any public method with 3+ kwargs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cosanlab/nltools](https://github.com/cosanlab/nltools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
