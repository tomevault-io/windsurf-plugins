---
trigger: always_on
description: This is the fast-start map of the PyGWalker repo for both human contributors and coding
---

# AGENTS.md — PyGWalker contributor & agent guide

This is the fast-start map of the PyGWalker repo for both human contributors and coding
agents. It explains how the project is put together, how to run it in **dev mode with live
frontend reload**, and where all the logs go. Read this first — it is written to save you
from re-deriving the architecture by grepping.

> Deeper references: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) (how it is built),
> [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md) (dev workflow + troubleshooting),
> [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md) (validation & CI).

---

## 1. What PyGWalker is (30-second model)

PyGWalker turns a pandas / polars / pyarrow dataframe into an interactive
[Graphic Walker](https://github.com/Kanaries/graphic-walker) UI inside notebooks, Streamlit,
and plain web servers. It has **two halves that ship together**:

- **Python package** (`pygwalker/`) — public API (`walk`, `render`, `table`, `Walker`,
  `to_html`), data parsing, and the transports that talk to the UI.
- **Frontend app** (`app/`, React + Vite) — the UI. It is compiled into JavaScript bundles
  that are checked into the wheel under `pygwalker/templates/dist/` and loaded by the Python
  side at render time.

The Python side never renders charts itself; it hands built JS + serialized data to a
notebook/browser and then answers data/spec requests over a message channel.

---

## 2. Repo map

| Path | What lives here |
|------|-----------------|
| `pygwalker/api/` | Public entry points. `adapter.py` picks jupyter vs webserver; `jupyter.py` = notebook dispatch; `walker.py` = the reusable `Walker`; `pygwalker.py` = the core `PygWalker`. |
| `pygwalker/services/` | Rendering + display. `anywidget_widget.py` (default transport), `render.py` + `templates/*.html` (iframe transport), `global_var.py` (runtime globals), `jupyter_display.py`. |
| `pygwalker/communications/` | Kernel⇄frontend transports: `anywidget_comm.py` (default), `hacker_comm.py` (iframe), `streamlit_comm.py`, `gradio_comm.py`, `reflex_comm.py`. `protocol.py` is the shared message schema. |
| `pygwalker/data_parsers/` | Dataframe/connector adapters (pandas, polars, pyarrow, SQL, spark…). |
| `pygwalker/templates/dist/` | **Build output** (git-ignored). The JS bundles the Python side loads. |
| `pygwalker/utils/` | Helpers: `frontend_assets.py` (locate/load bundles), `log.py` (logging), encoders. |
| `app/src/` | Frontend source. `index.tsx` = entry; `utils/communication.tsx` = transports; `dataSource/` = data ingest; `interfaces/comm.generated.ts` = **generated** protocol types; `store/` = MobX state. |
| `scripts/` | `dev.py` (dev orchestrator), `compile.sh` (build frontend), `local_ci.py` (mirror CI), `generate_comm_protocol_ts.py` (regenerate protocol types). |
| `tests/` | Python tests + `*.ipynb` notebooks run by `nbmake`. `app/tests/` holds Playwright smoke tests. |

---

## 3. How the two halves fit together (build & load model)

```
app/src/*  --(vite build)-->  pygwalker/templates/dist/*.js  --(read at runtime)-->  Python renders it
```

**Frontend build variants** (`app/vite.config.ts`, output to `pygwalker/templates/dist/`):

| Bundle | Built from | Loaded by |
|--------|-----------|-----------|
| `pygwalker-app.es.js` | `src/index.tsx` | **anywidget** transport (the default `pyg.walk` path) |
| `pygwalker-app.iife.js` | `src/index.tsx` | iframe transport / static `to_html()` |
| `dsl-to-workflow.umd.js` | `src/lib/dslToWorkflow.ts` | kernel-side DSL→workflow conversion |
| `vega-to-dsl.umd.js` | `src/lib/vegaToDsl.ts` | kernel-side Vega→DSL conversion |

`yarn build` builds all four (+ typecheck). `yarn build:app` builds only the two app
bundles (fast, no typecheck) — good for a quick manual rebuild, not for CI.

**The message protocol is generated, not hand-written.** Python Pydantic models in
`pygwalker/communications/protocol.py` are the source of truth. Running
`python scripts/generate_comm_protocol_ts.py` regenerates
`app/src/interfaces/comm.generated.ts`. **If you change `protocol.py`, regenerate and rebuild
the frontend.** Never edit `comm.generated.ts` by hand.

**Transports.** The default notebook transport is **anywidget** (`env='JupyterAnywidget'`).
`env='Jupyter'` / `env='JupyterWidget'` are deprecated aliases that are coerced to anywidget
and slated for removal in 0.7.0. Streamlit/Gradio/Reflex/web-server have their own transports.

---

## 4. First-time setup

Requires **Python 3.10+**, **Node.js 22.x**, **Yarn 1.x**.

```bash
# Python (editable install with dev extras)
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -e ".[dev]"

# Frontend deps + one full build so pygwalker/templates/dist/ is populated
cd app && yarn install && yarn build && cd ..
```

---

## 5. Dev mode: edit the frontend and see it live (anywidget HMR)

The default `pyg.walk(df)` uses the anywidget transport, which loads
`pygwalker-app.es.js` from disk. In dev mode we (a) rebuild that bundle on every source
change and (b) let anywidget hot-reload it into open widgets. **One command starts
everything and captures all logs:**

```bash
source venv/bin/activate
python scripts/dev.py
```

This launches, and tees the output of, two long-running processes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kanaries/pygwalker](https://github.com/Kanaries/pygwalker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
