---
trigger: always_on
description: >
---


# colab CLI — Agent Guide

CLI client for Google Colab runtimes. Manages runtime lifecycle, notebook sync, code execution, file transfer, and secrets via bash commands returning structured JSON.

Runtimes are full Linux VMs with a Python kernel, optionally with GPU or TPU accelerators. Shell commands are available through Python's `!` prefix or `subprocess`, but `exec` talks to the Python kernel — it is not a direct shell. `upload`/`download` transfer any file type.

Two commands require user browser interaction: `auth login` (one-time OAuth) and `ensure --drive` (one-time Drive consent). All other commands are fully non-interactive.

## Operating Model

| Command | Purpose |
|---------|---------|
| `ensure` | Create or reconnect to a runtime (GPU/TPU/CPU) |
| `pull` / `push` | Sync notebook source between local `.py` and remote `.ipynb` |
| `run` | Execute saved notebook cells on the remote kernel |
| `exec` | Run ad-hoc Python on the live kernel (ephemeral, not saved) |
| `status` / `ls` | Check runtime/kernel state, compute balance, list all notebooks |
| `diff` | Cell-level diff between local `.py` and remote `.ipynb` |
| `restart` | Restart kernel (clear Python state, keep runtime + packages) |
| `interrupt` | Cooperatively cancel running execution |
| `kill` | Release the runtime (destructive — clears all runtime state) |
| `upload` / `download` | Transfer files to/from the runtime (max ~384 MiB) |

**`run` vs `exec`:** `run` executes the notebook source (the remote `.ipynb`). Use `run --push` after editing to push and execute in one step. `exec` runs throwaway Python on the same kernel — useful for inspection and debugging, but it mutates kernel state without updating notebook source. If you used `exec` to define variables or imports, `restart` clears the drift.

**`exec` is Python, not shell.** The kernel is Python. Shell commands require the `!` prefix:
```bash
colab exec training "!pip install torch"       # correct — shell via Python
colab exec training "!ls /content/data/"       # correct
colab exec training "pip install torch"         # WRONG — Python SyntaxError
```

## First Run

```bash
colab auth login                  # opens browser — user must complete OAuth
colab auth status                 # verify: tier, compute units, eligible GPUs
colab ensure training --gpu t4    # allocate GPU runtime (blocks until ready)
# write training.py
colab run training --push         # push local .py then execute all cells
```

## Common Workflows

**Iterative development:**
```bash
# edit training.py
colab run training --push         # push + execute
# read results from stdout JSON → fix code → repeat
```

**Debugging a failure:**
```bash
colab run training --push         # cell fails → read traceback from data.results
colab exec training "print(x.shape)"  # inspect kernel state
# fix training.py
colab run training --push         # re-run
```

**File I/O:**
```bash
colab upload training data.csv data.csv       # upload to /content/data.csv
colab run training --push                      # training code reads data.csv
colab download training model.pt ./model.pt   # download result
```

**Runtime reclamation:** Colab reclaims idle runtimes after ~30 minutes. If this happens during `run` or `exec`, the WebSocket closes and the command returns an error. Recovery: `colab ensure <name> ...` to allocate a new runtime, then `colab push <name>` to restore from the local `.py` and cached `.ipynb`.

## Agent Rules

1. **Two commands need a browser.** `auth login` (one-time) and `ensure --drive` (one-time per runtime). Everything else is non-interactive.
2. **stdout is one JSON object when invoked as a subprocess** (non-TTY). Use `--json` to force JSON in all contexts. stderr carries progress, warnings, and consent prompts.
3. **Always `ensure` before other notebook commands.** `push`, `pull`, `run`, `exec` all require a live runtime.
4. **Use `run --push` after editing source.** This is the standard edit-execute cycle. `run` executes the remote notebook, not local `.py`.
5. **Use `exec` only for inspection/debugging.** It runs on the Python kernel — shell commands work via `!` prefix, not direct bash. Do not rely on `exec`-created state for reproducible runs.
6. **One command at a time per notebook.** No file locking — concurrent commands cause races.
7. **Names, not paths.** Always `colab run training`, never `colab run training.py`.
8. **Runtimes cost compute units.** Use `kill` when done. Check `auth status` for remaining balance and burn rate.

## Timeout Semantics

`--timeout` is a **per-cell client wait budget** — how long the CLI waits for each cell before returning control. It does NOT cancel or interrupt remote execution. A 5-cell notebook with `--timeout 300` could take up to 1500 seconds total.

If the timeout expires:
- CLI returns `TIMEOUT` (exit 6) with partial results if any
- Remote kernel may still be running — execution is not interrupted
- Check `colab status <name>` → `kernelState` to see if it's still busy
- Use `colab interrupt <name>` to explicitly cancel (cooperative, sends `KeyboardInterrupt`)
- Use `colab kill <name>` to release the runtime entirely (destructive)

## Output Contract


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brittlewis12/colab-cli](https://github.com/brittlewis12/colab-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
