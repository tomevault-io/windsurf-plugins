---
trigger: always_on
description: Every Linear task or subtask created by an agent must be fully documented at creation time with a state-of-the-art description: context, intent, acceptance criteria, validation plan, and handoff/deployment notes when relevant.
---

# AGENTS.md

## Linear Task Creation

Every Linear task or subtask created by an agent must be fully documented at creation time with a state-of-the-art description: context, intent, acceptance criteria, validation plan, and handoff/deployment notes when relevant.

Required creation metadata is non-negotiable: effort estimate, priority, and dependency relations (`blockedBy` / `blocks`) mapped from the plan, parent issue, or nearest sibling. If a connector cannot set one of those fields during creation, state the gap immediately and fill it through the next available Linear surface before handoff.


Contract for AI coding agents (Claude Code, Codex, and similar) working in this
repository. Humans: see [`CONTRIBUTING.md`](CONTRIBUTING.md) — the two are consistent;
this file states the parts an autonomous agent most needs up front.

> `AGENTS.md` is the only instruction file here; a tracked `CLAUDE.md` is a
> defect (CI guard rejects it). The loader keeps a read fallback for `CLAUDE.md`
> in other repositories — that is code behavior, not a file this repo tracks.
> (These governance files are themselves
> protected by the write guard — they are **refused** by the `commit_note` memory write
> path and can only be changed in an ordinary, reviewed PR.)

## The prime invariant

**Files are the source of truth; the index is a disposable, rebuildable projection of
the git tree.** Never treat the index as a database of record. A reindex must never be
able to lose a committed write. Everything in [`ARCHITECTURE.md`](ARCHITECTURE.md)
follows from this one line.

## Repo map — where things live

| Area | Path | What it is |
|---|---|---|
| Engine source | `src/hypermnesic/` | The Python package (details below) |
| Retrieval | `retrieve.py`, `index.py`, `embed.py`, `graph.py`, `converge.py` | Hybrid FTS5 + sqlite-vec search; read-time convergence |
| Write path | `commit_note.py`, `serialize.py`, `frontmatter_gate.py`, `audit_log.py` | The one git-first write, its guards, its gate, its log |
| Serving | `mcp_server.py`, `auth.py`, `auth_cloud.py` | Public OAuth `/mcp` + tailnet read companion |
| Local surface | `cli.py` | The engine-host-local `hypermnesic` CLI |
| Provisioning / config | `install.py`, `connect.py`, `config.py` | Roles, setup, env-driven config |
| Human surfaces | `capture.py`, `salience.py`, `nav_surface.py`, `think.py`, `propose.py`, `expand.py`, `sidecar.py`, `folders.py`, `generated.py` | Capture/triage, digest, navigation, thinking-mode, sidecar extraction |
| Tests | `tests/` | `pytest`, `--import-mode=importlib` |
| Gate scripts | `scripts/` | `check_version_consistency.py`, `license_scan.py`, `preflight_public_scan.py` |
| Plugin | `plugin/` | Claude Code / Codex plugin (OAuth-discovery, distribution-generic) |
| Companion | `obsidian-plugin/` | Read-only Obsidian companion (ships from a **separate** GPL-3.0 repo) |
| Benchmarks | `harness/` | LongMemEval harness + `harness/BENCHMARKS.md` |
| Docs | `docs/` | Start at [`docs/README.md`](docs/README.md) — it pins current truth |

## Build, test, and gates

Python ≥ 3.11, [uv](https://docs.astral.sh/uv/) for dependencies. The full gate set —
identical to CI's single `lint-test-license` job — is:

```sh
uv sync --extra dev
uv run ruff check .                                   # lint (line-length 100; E,F,I,UP,B)
uv run python scripts/check_version_consistency.py    # pyproject ↔ manifests ↔ __init__
uv run pytest                                         # full suite (offline, deterministic)
uv run python scripts/license_scan.py                 # zero AGPL/GPL/SSPL *dependency* gate
uv run python scripts/preflight_public_scan.py        # no operator secret/host in the public surface
```

All six must pass before a change is done. The suite runs offline and deterministic
(the OpenAI key is neutralized in tests; dense retrieval degrades to lexical).

## Working rules

- **Test-first.** No new production behavior without a failing test first. Tests live
  in `tests/`, run with `--import-mode=importlib`.
- **No "pre-existing" failures.** A red test is either fixed in your change or filed as a
  tracked issue — never dismissed or deleted.
- **Branch off `dev` and PR into `dev`; never commit to `dev` or `main` directly.**
  `dev` is the default branch and the baseline for all work. `main` is the **release
  branch** and receives `dev` only at release time — never a feature branch. Use a
  worktree per task when changes could conflict. Commit per logical unit with a
  conventional subject and a `Signed-off-by:` DCO line (`git commit -s`).
- **Permissive dependencies only.** Any new dependency must keep
  `scripts/license_scan.py` green (zero AGPL/GPL/SSPL). The gate is dependency-scoped;
  it does not constrain the engine's own (planned-AGPL) license.
- **Never echo secrets.** The OpenAI key, OAuth consent secret, and tokens are read
  from the environment / a gitignored `.env` only — never written to the index, the
  audit log, any output, or chat. `scripts/preflight_public_scan.py` enforces that no
  operator host/IP/token ships in the public surface — so **use placeholders**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonardsellem/hypermnesic](https://github.com/leonardsellem/hypermnesic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
