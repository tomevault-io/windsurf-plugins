---
trigger: always_on
description: This file is a **portable context for AI coding agents** (Claude Code, etc.) and
---

# WeightsLab — agent context for users & debugging

This file is a **portable context for AI coding agents** (Claude Code, etc.) and
the humans driving them. Its job is to let you — or an agent helping you —
**install, configure, run, and debug WeightsLab and Weights Studio** without
having to reverse-engineer the system first.

It deliberately covers only the two shipped repositories:

- **weightslab** — the Python backend / core (training instrumentation, data
  ledger, gRPC service, the shared proto).
- **weights_studio** — the browser frontend (the studio UI that inspects and
  edits a *running* experiment).

> File/line references drift as the code evolves — treat them as starting points
> and verify against the current source before relying on them. Environment
> variable names and defaults are the most stable thing here; when in doubt the
> authoritative reference is `weightslab/docs/configuration.rst`.

---

## 0. How to load this guide into Claude Code

So an agent actually *has* this context when you ask it for help:

- **Working inside a checkout of the repo** (`git clone`): this guide is
  committed as `AGENTS.md`; the repo keeps a gitignored `CLAUDE.md` copy of it at
  the root so Claude Code auto-loads it every session. Nothing to do. (Claude
  Code also loads `~/.claude/CLAUDE.md` global memory and any parent-dir
  `CLAUDE.md`.)
- **You only ran `pip install weightslab`** (no checkout — the package lives in
  `site-packages`): absolute `@import` paths are fragile because the path
  changes per venv/OS. The robust pattern is a small **skill** that locates the
  installed file at runtime. Create `~/.claude/skills/weightslab/SKILL.md`:

  ```yaml
  ---
  name: weightslab
  description: Load the WeightsLab debugging & configuration guide when helping with weightslab or weights_studio problems (connection, TLS, env vars, training hangs, rendering).
  ---
  !`python -c "import weightslab, os; print(open(os.path.join(os.path.dirname(weightslab.__file__), 'AGENTS.md')).read())"`

  Use the guide above to diagnose the user's weightslab / weights_studio issue.
  ```

  Then run `/weightslab` (or let Claude auto-invoke it). This requires the guide
  to be **shipped as package data** inside the installed package (see §7); the
  copy at the repo root is for contributors working in a checkout.
- **Quick-and-dirty:** copy this file to `~/.claude/WEIGHTSLAB.md` and add
  `@~/.claude/WEIGHTSLAB.md` to your `~/.claude/CLAUDE.md`.

---

## 1. What it is and how the pieces connect

A user wraps their own PyTorch training script with WeightsLab so a running
experiment becomes inspectable/editable; Weights Studio is the UI for that.

**Wire path (the thing that breaks most often):**

```
Browser  →  weightslab start :8080 (grpc-web → grpc proxy)  →  Python gRPC servicer  →  training loop
```

- `weightslab start` is a pure-Python HTTP server that serves the bundled SPA
  and translates grpc-web (browser) to raw gRPC (backend). No Docker, no Envoy.
  If `weightslab start` is not running, the browser has no UI to load.
- The gRPC servicer and the training loop run in the **same process, different
  threads**, coordinated by locks in
  `weightslab/weightslab/components/global_monitoring.py`.
- One proto is the single source of truth:
  `weightslab/weightslab/proto/experiment_service.proto`.

---

## 2. Install & run (the happy path)

```bash
pip install weightslab
```

In your training script:

```python
import weightslab as wl
# wrap your objects so the studio can see/edit them (see §3), then:
wl.serve(serving_grpc=True, serving_cli=True)   # background threads, same process
# ... your training loop ...
wl.keep_serving()                                # keep the process alive for the UI
```

Then start the UI in another terminal and open it in a browser:

```bash
weightslab start   # serves at http://localhost:8080 by default
```

Working starting points live in
`weightslab/weightslab/examples/{PyTorch,Lightning,Usecases}/<usecase>/`
(each is a `main.py` + `config.yaml`) — find the closest example and mirror it.

UI deployment details (port, TLS, certs) are documented in
`weightslab/docs/weights_studio.rst`. TLS is opt-in: run `weightslab se` once,
then `weightslab start --certs`.

---

## 3. The integration API (`import weightslab as wl`)

How a user's script plugs in. Wrap each training object with
`wl.watch_or_edit(obj, flag=...)`; the returned tracked proxy is registered in
the global ledger (`weightslab/weightslab/backend/ledgers.py`,
`GLOBAL_LEDGER` — the hub everything reads/mutates through).

- `flag="hyperparameters"` (dict), `flag="model"` (nn.Module, `device=…`),
  `flag="optimizer"`, `flag="data"` (Dataset → tracked DataLoader: `loader_name`,
  `batch_size`, `is_training`, `collate_fn`, …), `flag="loss"` (a
  `reduction="none"` criterion, called with `(preds_raw, targets, batch_ids=ids,
  preds=preds)`), `flag="metric"`.

Conventions that matter for correctness:

- Wrap the train step in `with guard_training_context:` and eval in
  `with guard_testing_context:` (from
  `weightslab.components.global_monitoring`). This is how pause/resume and
  train/test separation work — **skip it and pause/resume or stats will misbehave.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GrayboxTech/weightslab](https://github.com/GrayboxTech/weightslab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
