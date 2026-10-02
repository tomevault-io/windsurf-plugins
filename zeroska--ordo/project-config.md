---
trigger: always_on
description: This repo ships **portable OSINT skills** (`WebPivot`, `IntelAnalysis`, `IntelGraph`,
---

# intelligence_assist — contributor rules

This repo ships **portable OSINT skills** (`WebPivot`, `IntelAnalysis`, `IntelGraph`,
`BinaryPivot`) plus shared tooling under `tools/`. The skills are imported onto other
machines and used by other people. Treat everything tracked here as **public-facing**.

## RULE 1 — Never put case / investigation data into a skill (CRITICAL)

Skills are **code + tradecraft only**. An investigation's data NEVER goes into a
`SKILL.md`, a workflow `.md`, a tool docstring/comment, a test fixture, or any tracked
file. This includes — in prose, comments, examples, fixtures, or hardcoded logic:

- **Real people / operators** — names, aliases, emails, phone / Zalo / Telegram / Messenger handles.
- **Real target infrastructure** — case domains, IPs, wallets, ASNs, hostnames.
- **Real owner artifacts** — actual GA4/GTM/UA IDs, ahrefs/GSC tokens, favicon/DOM hashes tied to a case.
- **Case identifiers** — `CASE-YYYY-NN` IDs, case-folder names, per-case hardcoded paths.
- **Operator PII / attribution** of any kind, even as a "worked example".

Investigation data lives ONLY in the git-ignored stores: `cases/`, `knowledge/`,
`MEMORY/`, `.env`, and the operator registry. It is never committed and never referenced
by identifier inside a skill.

## RULE 2 — Register every new tool/skill with the MCP (so Claude Code can use it)

When you add a **tool** or a **skill**, publish it through the one typed surface both front-ends
share (`harness/tools.py` → the SDK `orchestrator.py` **and** the stdio `harness/mcp_server.py`,
which auto-discovers every `@tool`). Do NOT leave a new capability reachable only as a raw
`python3 …` bash line.

- **New CLI tool** (`WebPivot/tools/*.py`, `tools/*.py`): wrap it as an `@tool(name, description,
  {params})` in `harness/tools.py`. `mcp_server.py` discovers it automatically (no second edit) and
  it appears to Claude Code via the repo-root `.mcp.json` server `intel`. Keep the description one
  tight paragraph — it is context cost paid on every SDK phase (see below).
- **New mode of an existing tool** (e.g. IPPivot is just `pivot_extract.py` with a bare-IP source):
  no new `@tool` — extend the existing tool's description so the model knows the new input/flag.
- **New skill** (`WebPivot`, `IntelAnalysis`, …): add its `SKILL.md`, symlink it into
  `~/.claude/skills/`, and if it exposes a scriptable step, surface that step as an `@tool` too.
- **Smoke-check registration:** `WebPivot/.venv/bin/python3 harness/mcp_server.py` then send a
  `tools/list` JSON-RPC — the new tool must be listed. In Claude Code, confirm with `/mcp`.

## RULE 3 — Separate DATA from LOGIC: reference lists live in JSON, never in code

An analyst must be able to tune a denylist, threshold or lookup table **without editing Python
and without a redeploy**. Code holds the *matching logic*; the values it matches against are data.

- **Any list, map or threshold an analyst may reasonably want to extend goes in a JSON file** —
  denylists/allowlists (managed DNS, parking hosts, privacy-proxy contacts, noise phones), scoring
  thresholds, ASN/CIDR tables, brand/keyword sets, provider registries. If you catch yourself
  appending a literal to a Python tuple/set/dict, it belongs in JSON instead.
- **Where it lives:** `<module>/references/<name>.json` — e.g. `WebPivot/references/cdn_ranges.json`,
  `IntelAnalysis/references/risk_indicators.json`, `tools/kb/references/noise_filters.json`.
- **Shape:** a top-level `_comment` explaining the file, then one object per group with its own
  `_comment` and a `values` array (or named scalars for thresholds). The `_comment` keys are the
  analyst's documentation — write them for a human who has never read the code. Keys beginning
  with `_` are ignored by loaders.
- **Loading:** use the shared loader — `wp_refs.py` (WebPivot), `kb_refs.py` (`tools/kb`),
  `bp_refs.py` (BinaryPivot), `ig_refs.py` (IntelGraph):
  `load_ref(ref_path(__file__, "<name>.json"), _FALLBACK)`. It
  **falls back to your minimal embedded default on missing/malformed/incomplete input and warns
  on stderr**. Never fail open silently — a filter that quietly returns `False` everywhere
  manufactures false clusters, which is worse than crashing. Keep the existing module-level
  constant names so importers don't break. The four loaders are **byte-identical copies on
  purpose** (each skill is imported standalone, so it can't depend on a repo-root package) with
  **distinct module names on purpose** (`tools/kb` and `WebPivot/tools` both land on `sys.path`
  in the same process, so a shared `refs.py` would collide); `tests/test_references.py` asserts
  they stay in sync.
- **One group, one owner.** If two modules match the same values, they read the same JSON group —
  never re-paste the list. That duplication is what let the registrant-noise denylists drift
  across six modules before this layer existed.
- **Normalise on load**, so analysts can enter values in any reasonable format (a phone as
  `+354.421 2434` or `3544212434`; a host with or without a trailing dot).
- **Test the data file itself** in `tests/test_references.py`: it asserts every `references/*.json`
  parses and is documented (`_comment` at the top and per group), that each consumer's loaded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zeroska/Ordo](https://github.com/Zeroska/Ordo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
