---
trigger: always_on
description: Portable context for AI coding agents (and their humans) to **install, run,
---

# WeightsLab — agent context for users & debugging

Portable context for AI coding agents (and their humans) to **install, run,
integrate, and debug WeightsLab / Weights Studio** without reverse-engineering
the system first. Covers two repos: **weightslab** (Python backend — training
instrumentation, data ledger, gRPC service, shared proto) and **weights_studio**
(browser frontend that inspects/edits a *running* experiment).

> File/line refs drift — verify against current source. Env var names/defaults
> are stable; authoritative reference is `weightslab/docs/configuration.rst`.

---

## 0. Loading this guide into Claude Code

- **Repo checkout:** committed as `AGENTS.md`; a gitignored `CLAUDE.md` copy at
  the root gets auto-loaded every session. Nothing to do.
- **`pip install weightslab` only** (no checkout): absolute paths are fragile
  across venvs/OS. Use a skill that locates the installed copy at runtime —
  `~/.claude/skills/weightslab/SKILL.md`:

  ```yaml
  ---
  name: weightslab
  description: Load the WeightsLab debugging & integration guide for weightslab/weights_studio problems (connection, TLS, env vars, training hangs, rendering, wl.* integration).
  ---
  !`python -c "import weightslab, os; print(open(os.path.join(os.path.dirname(weightslab.__file__), 'AGENTS.md')).read())"`

  Use the guide above to diagnose or implement the user's request.
  ```

  Requires the guide shipped as package data (`weightslab/weightslab/AGENTS.md` — see §7).
- **Quick-and-dirty:** copy this file to `~/.claude/WEIGHTSLAB.md`, `@`-import it
  from `~/.claude/CLAUDE.md`.

---

## 1. What it is, how the pieces connect

A user wraps their PyTorch training script with WeightsLab so a running
experiment becomes inspectable/editable; Weights Studio is the UI.

```
Browser  →  weightslab start :8080 (grpc-web → grpc proxy)  →  Python gRPC servicer  →  training loop
```

- `weightslab start`: pure-Python HTTP server, serves the bundled SPA and
  translates grpc-web↔gRPC. No Docker, no Envoy. Not running ⇒ no UI to load.
- gRPC servicer and training loop share **one process, different threads**,
  coordinated by locks in `weightslab/weightslab/components/global_monitoring.py`.
- Proto is the single source of truth: `weightslab/weightslab/proto/experiment_service.proto`.

---

## 2. Install & run

```bash
pip install weightslab
```

```python
import weightslab as wl
# wrap objects so the studio can see/edit them (§3), then:
wl.serve(serving_grpc=True, serving_cli=True)
# ... training loop ...
wl.keep_serving()   # keep process alive for the UI
```

```bash
weightslab start   # http://localhost:8080 by default
```

For a new script, pick the closest match in
`weightslab/weightslab/examples/{PyTorch,Lightning,Ultralytics,Usecases}/<usecase>/main.py`
via the decision table in §3.9, then copy its `wl.*` calls — §3 documents that
whole API surface (reactive signals, group signals, the Ultralytics mixin,
etc. aren't in the `.rst` docs; the examples are the primary source).

TLS/UI deploy details: `weightslab/docs/weights_studio.rst`. TLS is opt-in:
`weightslab se` once, then `weightslab start --certs`.

---

## 3. The integration API (`import weightslab as wl`)

How to wire a new training script correctly with no docs access — every verb
and kwarg here is real, taken from a shipping example under
`weightslab/weightslab/examples/` and checked against `weightslab/src.py`.

### 3.1 Lifecycle

```python
import weightslab as wl

wl.watch_or_edit(..., flag=...)                            # register objects (§3.2)
wl.serve(serving_grpc=True, serving_cli=True)    # background threads, same process
wl.start_training(timeout=3)                     # let UI/CLI attach before stepping
# ... training loop, guarded (§3.5) ...
wl.keep_serving()                                # block so the process/UI survives
```

- Register every object with `watch_or_edit` **before** `wl.serve`, using the parameter flag to define which object category it is.
- `timeout=0` skips the pre-start wait entirely.
- Skip `keep_serving()` for a script that should exit after writing a report
  (`Usecases/*signals*` examples); include it otherwise.
- Tabular examples (`wl-fraud-detection`, `wl-ads-recommendation`) pass only
  `serving_grpc=` to `serve` — no `serving_cli`.

### 3.2 `wl.watch_or_edit(obj, flag=..., **kwargs)`

Registers/wraps `obj` in the global ledger (`backend/ledgers.py`,
`GLOBAL_LEDGER`) and returns a live proxy. `flag` matches by substring
(case-insensitive).

| flag | wraps | key kwargs |
|---|---|---|
| `"hyperparameters"` | plain `dict` | `defaults=parameters`, `poll_interval=1.0`, optional `name=` |
| `"model"` | `nn.Module` | `device=`; `compute_dependencies=False` (skip arch-op dependency graph when not editing architecture); `forced_model_wrapping=True` (Ultralytics only — load current object, not a checkpoint) |
| `"optimizer"` | `torch.optim.Optimizer` | none typically; build from the **watched** model's `.parameters()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GrayboxTech/weightslab](https://github.com/GrayboxTech/weightslab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
