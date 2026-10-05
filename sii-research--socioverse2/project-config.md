---
trigger: always_on
description: > **Building a NEW study / compatible component** (e.g. migrating HiSim, ElectionSim, or writing
---

# SocioVerse2 — usage guide for Claude

> **Building a NEW study / compatible component** (e.g. migrating HiSim, ElectionSim, or writing
> new providers)? Read **`CLAUDE-dev.md`** + **`README-dev.md`** instead — this file is for *using*
> the framework to run studies, not extending it.

A **longitudinal, LLM-native social-simulation runtime**. Core idea = Lewin's field theory
**B = f(P, E)**: a FIXED population pool `P` (persistent agent ids) under a DYNAMIC environment
`E` (two axes: physical/information × macro/local), driven step-by-step so the *same* agents are
tracked over time (panel data) — the differentiator vs AgentSociety / OneSim.

## Environment & how to run

ALWAYS use the Python environment SocioVerse2 is installed in (Python 3.11+; see `README.md`):

```bash
PY=python
$PY -m pip install -e ".[dev,viz]"      # from the repository root; add extras as needed (llm, workbench, lit, abm, chicago)
$PY -m pytest tests -q                  # all tests; studies that wrap a companion repo skip when it is absent
$PY scripts/demo_offline.py             # opinion_diffusion end to end as the local study opinion_diffusion_demo, no key (llm_kind scripted)
$PY studies/chicago_schelling/run_demo.py --steps 3 --intervention-step 2   # live Chicago demo (needs the SocioVerse-ABM sibling + [chicago])
```

- **Secrets**: the LLM key is read from env `SV_LLM_API_KEY` (or `OPENAI_API_KEY`), with a gitignored
  `.env` here as fallback (see `.env.example`). NEVER hard-code keys.
- **No-token mode**: set `llm_kind: "scripted"` in a from-scratch study's `simulation.json` decision_args;
  for chicago, pass `llm_client=DeterministicLLMClient()` to `build_chicago_simulator` to dry-run
  the plumbing without spending API.

## The workflow you run — `init · build-model · build-environment · build-population · run · report · iterate`

Claude-Code skills (in `.claude/skills/`). Each writes ONE validated artifact into
`studies/<id>/`, then **by default pauses for you to inspect / edit / redo before the next** — so you
can change course or roll back at any gate. Tell it how far to run (e.g. "build everything, prompt me
before `/sv-run`", or "do the whole pipeline") and it follows your flow, chaining the stages while
still surfacing each artifact as it passes.

| skill | what it does | writes |
|---|---|---|
| `sv-init` | **intent checklist first** (Step −1 intent checklist: seven fixed dimensions to align on what the task is, rendered live in the cockpit; the user may delegate and take the defaults), then **route the query** against already-adapted studies (reuse vs build-new) and parse it into a study **+ grounding bootstrap** (real-world anchors via search) | `intent.json` + `study.yaml` + `grounding/grounding.json` |
| `sv-build-model` | *(Path B only)* implement a from-scratch study's four abc natively on Core | `model.py` |
| `sv-build-environment` | build E (layers + interventions + broadcasts) | `environment/environment.json` |
| `sv-build-population` | build P **and instantiate the agents** (materialize the pool now, not at run start) | `population/population.json` + `population/roster.jsonl` |
| `sv-run` | run the longitudinal simulation | `trajectory/study.duckdb` + `metrics_history.json` |
| `sv-report` | render the time-step change | `reports/report.md` + figures |
| `sv-lit` | **survey real related work** (multi-source retrieval via the vendored `paper-search-pro` engine, per-paper relevance notes; never fabricates a reference) — any point after sv-init; **`sv-paper` requires it** | `literature/literature.json` + `references.bib` |
| `sv-paper` | **draft the full academic paper** from the study's own artifacts (report/DuckDB/grounding/literature) — after sv-report. Prose craft is delegated to the vendored `research-paper-writing` skill (see below) | `paper/paper.md` |
| `sv-iterate` | **change an EXISTING study** (the only post-run entry): the version / branch gate → re-enter at the affected stage | `versions.json` + `versions/vN/` snapshots |

To run a skill, read its `.claude/skills/<name>/SKILL.md` and follow it. Skills are discovered at
**session start**, so open the repository root as the project and use a fresh session.

**Literature search engine**: `.claude/skills/paper-search-pro/` is also a vendored third-party skill (Apache-2.0,
O0000-code/paper-search-pro) — a joint search over five sources (OpenAlex / Semantic Scholar / CrossRef / PubMed / arXiv)
plus native-Chinese sources (NSSD, yiigle), with federated dedup and a saturation signal. `sv-lit` drives retrieval through its **agent channel**
(`references/agent_mode.md`) and itself only owns "what these papers mean for this study". It **needs pip dependencies**
(pyalex etc., `pip install -e ".[lit]"`); when they cannot be installed, `sv-lit` automatically falls back to the zero-dependency
OpenAlex path in `skills/sv_lit.py` — details in its `VENDOR.md`. **`/sv-paper` now hard-requires `/sv-lit` to have run first**:
without `literature/literature.json`, run it first; writing related work from memory is exactly what this pipeline exists to prevent.

**The paper-writing "craft layer"**: `.claude/skills/research-paper-writing/` is a third-party skill vendored from GitHub

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sii-research/SocioVerse2](https://github.com/sii-research/SocioVerse2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
