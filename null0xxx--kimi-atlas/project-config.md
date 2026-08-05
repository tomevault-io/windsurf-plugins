---
trigger: always_on
description: Read this first in any session touching this repo. It is the durable, fact-checked map of
---

# AGENTS.md — kimi-atlas project memory

Read this first in any session touching this repo. It is the durable, fact-checked map of
what exists, how to verify it, and what is still open. For depth, follow the links to
[`references/`](references/) — especially [`references/architecture.md`](references/architecture.md),
[`references/atlas-weave.md`](references/atlas-weave.md), [`references/rubric.md`](references/rubric.md),
[`references/skill-registry.md`](references/skill-registry.md), and the plan docs under
[`docs/superpowers/plans/`](docs/superpowers/plans/).

## What this is

**kimi-atlas** — a many-agent, quality-calibrated orchestrator plugin for Kimi Code with **115
vendored official skill packages** built in. Public repo: <https://github.com/null0xxx/kimi-atlas>
(v1.3.0, MIT). Install: `/plugins install https://github.com/null0xxx/kimi-atlas` (managed copy at
`~/.kimi-code/plugins/managed/kimi-atlas`); from source: `./scripts/install.sh`
(installs to `~/.kimi-code/plugins/kimi-atlas`).

Four layers, all first-party:

- **atlas** (`skills/atlas/SKILL.md`) — single-change core: deterministic
  `INIT → INTENT_CAPTURED → [CLARIFY] → TRIAGED → GROUNDED → CODED → VERIFIED → [REFINE]* → OUTPUT`
  state machine; 6-lens verification harness (deterministic `runcheck`/`lint`/`reqcoverage`/
  `pathcheck`/`astlens` floor + 3 isolated adversarial critics); **no LLM ever computes pass/fail**
  (`verdict.merge`/`gate` are pure). Never auto-applies; human gates only.
- **ATLAS-WEAVE** (`skills/atlas-weave/SKILL.md`) — multi-agent meta-machine: file-disjoint
  plan-DAG, ≤3 concurrent inner atlas runs, combined-tree differential integration.
- **The agentic backbone (Graph + Loop + Verification)** — wraps the pure core, never replaces it
  (merged `da90f6c`, 6-lens-hardened `27→0`): **ContextGraph** (`scripts/contextgraph.py`) — pure
  read-time projection over the ledger + `hooks.jsonl`, injected as SAFE-2 DATA into the CODED coder
  packet (recomputed each REFINE; a hint, never a gate); **`scripts/ctxevents.py`** records
  tool_call/error events to `hooks.jsonl` (never `log.jsonl`); **`scripts/fsm.py`** — `legal_transition`
  derived from `ctxstore.STAGES` + one declared `REFINE→CODED` edge; **`scripts/rollback_driver.py`** —
  two-phase forward-only rollback (pure `sanctioned_rollback` + monkeypatchable git seam, worktree-only,
  append-only ledger); **`scripts/safewrap.py`** — the single canonical SAFE-2 wrapper; **`scripts/astlens.py`**
  — `ast` syntax/lint lens folded into VERIFIED; **`scripts/rubric.py`**/**`scripts/frontmatter.py`** —
  single-source rubric vocab / shared BOM+CRLF frontmatter primitive.
- **The skill system** — 115 vendored skill packages + registry/selector (below).

## Commands (the daily five)

```bash
make ci               # THE gate: strict naming + unit tests + inventory-drift + shell syntax
make test             # the full unit-test suite (python3 -m unittest discover -s tests -v)
make skill-registry   # rebuild references/skill-registry.json from the extracted skills/ tree
make skills-extract   # re-extract vendored packages + --verify against the sha256 manifest
make negative-gate    # red-team fixtures: good→OK, each bad_*→UNVERIFIED
```

`make ci` mirrors `.github/workflows/check.yml` (Python 3.12). Everything must stay green.

## Non-negotiable conventions (any edit must match)

- **Python:** stdlib-only 3.12, `from __future__ import annotations`, pure cores + thin I/O
  "hands", long module docstrings citing invariants, CLI = `main(argv=None) -> int` +
  `sys.exit(main())`, plugin root via `pathlib.Path(__file__).resolve().parents[1]` + sys.path shim.
- **Output idiom:** `sys.stdout.write` / `sys.stderr.write` in the `skill*` modules — the atlas
  harness lints changed files for `print(` as a debug token (repo's older CLIs use `print()`).
- **Tests:** stdlib `unittest` only, `tests/test_<module>.py` per `scripts/<module>.py`,
  tempfile fixture trees, in-process `main()` via `redirect_stdout/stderr`, behavior AND
  failure-path assertions; `TestMainRealRepo`/`TestCommitted*` classes pin the real tree.
- **Doc gates:** new `.md` = lowercase kebab-case (exempt basenames: `README.md`, `SKILL.md`,
  `LICENSE`, `Makefile`, `PLAN.md`, `AGENTS.md`) AND individually markdown-linked from
  `references/*.md` or `README.md` (a directory link does not count). A `skills/` dir containing
  `SKILL.md` is a self-contained vendored package — exempt via `scripts/skillpkgs.walk_markdown`.
- **Backticked path citations** in changed text must exist on disk (harness `pathcheck` scans
  `-`/`+`/context diff lines); use the `.atlas/<run_id>/…` placeholder form for run artifacts.
- **Determinism:** generated artifacts are sorted, stable-keyed, timestamp-free; writers follow
  validate→audit→write and never persist partial state.

## The skill system (v2, manifest-anchored)

- `skills/<name>/` — 115 vendored official packages (712 files, byte-identical to their source
  zips; 2 duplicate zips coalesced 117→115) + 3 first-party orchestrator skills. Platform-
  registered via `.kimi-plugin/plugin.json` (`"skills": "./skills/"`).
- `references/skills-manifest.json` — sha256 anchor for every vendored file;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [null0xxx/kimi-atlas](https://github.com/null0xxx/kimi-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
