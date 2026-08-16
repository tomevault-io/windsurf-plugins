---
trigger: always_on
description: Context for agents working in this repo. Read `AUDIT-FIXES.md` next if you were sent here to fix defects.
---

# CLAUDE.md — FERAL / ASOS

Context for agents working in this repo. Read `AUDIT-FIXES.md` next if you were sent here to fix defects.

## What this is

FERAL is a local-first AI runtime that users install on their own machine (macOS 13+ / Linux, Python 3.11+). It orchestrates LLM providers, keeps a 4-layer memory store, drives hardware over a WebSocket protocol (HUP), and emits server-driven UI ("Gen-UI" / SDUI) that clients render.

Version `2026.8.8`. Public beta. Single-user local deployment is the only supported target.

## Layout

| Path | Language | What it is |
|---|---|---|
| `feral-core/` | Python | The brain. ~170k production lines, ~130k test lines. Everything below is under here unless stated. |
| `feral-core/agents/` | Python | Orchestrator (`orchestrator.py`), LLM router (`llm_provider.py`, hand-rolled httpx to 16 providers) |
| `feral-core/api/` | Python | FastAPI app. `server.py` is the entrypoint, `state.py` holds the singleton |
| `feral-core/memory/` | Python | aiosqlite store, embeddings, knowledge graph, CRDT federated sync |
| `feral-core/models/protocol.py` | Python | **Canonical wire schemas.** `HUP_VERSION` lives at line 15 |
| `feral-core/cli/` | Python | `feral setup / start / doctor / memory / sync / access` |
| `feral-core/voice/`, `perception/` | Python | STT/TTS routing, VAD, wake word, sensor fusion |
| `feral-client-v2/` | React (JSX) | Current web client. `feral-client/` is the superseded v1 |
| `feral-nodes/` | Python, TS, Swift, Kotlin | Device SDKs and daemons. `HUP_SPEC.md` is the protocol source of truth |
| `feral-registry/`, `feral-relay/` | Python | App registry; NAT-traversal relay |
| `desktop/` | Rust (Tauri) | Experimental shell. Bundles its own CPython + a copy of `feral-core` and spawns `python -m api.server` against them; see "The interpreter" below |

## Commands

```bash
make dev                                  # build pinned .venv, install feral-core[all,dev] + client deps
make test                                 # cd feral-core && python -m pytest tests/ -v
make serve                                # feral serve
make doctor                               # feral doctor — reports real runtime state
```

`make dev` installs into `.venv/` at the repo root, built from `.python-pin`, and every other target routes through it. It needs `uv >= 0.12` and fetches a repo-local one if your machine has none. See below.

CI equivalents (what actually gates a PR):

```bash
cd feral-core && ruff check --select=E,F,W --ignore=E501,E402,F401,W291,W293 .
cd feral-core && pip install --constraint requirements.lock -e ".[all,dev]"
cd feral-core && python -m pytest tests/ --cov --cov-fail-under=50
```

`make lint` does **not** lint — it runs pytest and prints a note. Use the ruff line above.

## The interpreter: pinned for dev, bundled for users

This is the single most load-bearing environment fact in the repo. Read it before you debug anything that looks like "memory is broken on my machine".

### The two SQLite features, and why they are separate

FERAL's SQLite needs two independent build-time features. Stock interpreters routinely ship one and not the other, and the consequences are not symmetric.

| Interpreter | SQLite | FTS5 | loadable extensions |
|---|---|---|---|
| pyenv 3.11.11 (macOS default build) | 3.51.0 | **yes** | no |
| python-build-standalone 3.11.13 | 3.49.1 | **no** | yes |
| **python-build-standalone 3.11.15 (pinned)** | 3.53.1 | **yes** | **yes** |

**FTS5 is required.** `memory/store.py` and `memory/knowledge_graph.py` create five `CREATE VIRTUAL TABLE ... USING fts5` tables during construction. Without it the store does not degrade, the brain does not start. It used to die as `sqlite3.OperationalError: no such module: fts5` pointing at a triple-quoted SQL string; `memory/sqlite_features.require_fts5` now raises `SQLiteFeatureError` naming the interpreter, the SQLite version and the fix, before any DDL runs so no half-created database is left behind.

**Loadable extensions are optional.** They gate `sqlite-vec`. pyenv's macOS default omits `--enable-loadable-sqlite-extensions`, so `sqlite3.Connection` has no `.enable_load_extension` at all, while `pip install sqlite-vec` and `import sqlite_vec` both still succeed. `sqlite_vec_available()` returns False, logs at INFO, and the vector leg runs over numpy. Per F-17 that numpy path is the **faster** of the two (0.46ms vs 7.08ms for top-5 over 12k 384-dim vectors), so this costs resident memory and nothing else. Never prescribe an interpreter rebuild as a remedy for it.

**Neither feature implies the other.** 3.11.13 has extensions and no FTS5; pyenv 3.11.11 has FTS5 and no extensions. Anything that checks one and infers the other is wrong. `memory/sqlite_features.py` is the single place both are probed, and `feral doctor` renders them as two separate rows with two different severities (`SQLite FTS5` is a `_fail`, `SQLite loadable extensions` is an `_info`).

### Development: `.python-pin` and `make dev`

From a clean clone, one command:

```bash
make dev
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FERAL-AI/FERAL-AI](https://github.com/FERAL-AI/FERAL-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
