---
trigger: always_on
description: Instructions for coding agents (Claude Code, Codex, Cursor, Aider, and anything else that edits
---

# AGENTS.md

Instructions for coding agents (Claude Code, Codex, Cursor, Aider, and anything else that edits
this repository unattended). Human contributors want [`CONTRIBUTING.md`](./CONTRIBUTING.md) — it
covers the CLA, the review process and the AI-disclosure policy, and it is the authority on all
three. This page is the operational half: the repository's shape, the gates, and the specific
things that are easy to get wrong here and expensive to get wrong.

> **Are you here to *use* the framework rather than change this repository?** You want
> [`docs/agent-briefing.md`](./docs/agent-briefing.md) instead — the mental model, every Provider
> available at each Stage, the Pipeline YAML contract, the CLI, the output schema and the retrieval
> wiring. [`llms.txt`](./llms.txt) indexes the rest. This page is for agents **editing this
> codebase**: the gates, the contracts, and the traps.

Two ground rules before anything else:

1. **The tree is the authority.** Every claim below was read out of a file at the path named. If a
   command here disagrees with the repository, the repository is right and this page is a bug —
   fix it in the same change.
2. **This repository rejects plausible-sounding claims.** Documentation that makes an assertion
   about the code is machine-checked by ordinary pytest modules — see item 4 of § *Six things that
   will burn you*. A confident sentence you cannot substantiate will not merge; it will turn CI red.

---

## What this is

A model-agnostic, cloud-agnostic Python framework that turns messy enterprise documents into
AI-ready data: parse → chunk → screen → extract entities and relations → redact PII → resolve →
knowledge graph → export. The framework names **Capabilities** (narrow typed interfaces) and lets
**Providers** fulfil them; the core never names a model. [`CONTEXT.md`](./CONTEXT.md) is the
canonical glossary — use its terms exactly (Stage, Runner, Capability, Provider, Provenance,
Evidence, Quarantine) and avoid the synonyms it bans.

---

## Repository shape

A [uv](https://docs.astral.sh/uv/) workspace. `[tool.uv.workspace] members = ["packages/*"]` in the
root `pyproject.toml`; **34 packages** under `packages/`, each an independently publishable
distribution.

**One of them is deliberately NOT a workspace member.** `packages/latence-gliner25` pins a
`gliner2` major disjoint from the one the other Provider packages pin, and a uv workspace is ONE
resolved environment — `uv lock` refuses the set, correctly — so it is listed under
`[tool.uv.workspace] exclude` (ADR-0064). Everything that works by directory (ruff,
`scripts/typecheck.sh`, pytest, the contract reference, the environment specs) covers it unchanged;
three things had to be taught about it, and all three are gated: the release builds by **path**
(`uv build "$pkg"`, not `--package <name>`), the package carries its **own committed `uv.lock`**,
and `scripts/generate-sbom.sh` folds that lock into the SBOM union.

```
packages/latence-<family>-<name>/     33 of these
├── pyproject.toml                    deps, classifiers, the entry-point registration
├── LICENSE                           must match the root LICENSE
├── README.md                         this distribution's PyPI long_description
├── src/latence_<family>_<name>/      note: underscores here, hyphens in the directory name
│   ├── __init__.py
│   ├── provider.py
│   └── py.typed
└── tests/
    ├── conftest.py
    └── test_*.py
```

**The directory is hyphenated, the module is underscored.** `packages/latence-parser-pdfplumber/`
holds `src/latence_parser_pdfplumber/`. This is not cosmetic — it is the direct cause of the mypy
rule in § *Six things that will burn you*.

Everything else at the top level:

| Path | What it is |
|---|---|
| `packages/latence-core/` | contracts, Capability protocols, the plugin registry, the local Runner, Storage, the CLI, the conformance suite. Near-zero-dep and pure-Python **by policy** (ADR-0016) |
| `packages/latence-retrieval/` | the stateless query-time library. A separate downstream layer, not a Stage; it registers under a different entry-point group |
| `benchmark/` | measurement harnesses. `benchmark/s5/` is the retrieval matrix, `benchmark/serving/` the live-engine lane, `benchmark/sota-campaign/` the pipeline campaign |
| `scripts/` | `verify-local.sh` (the merge gate), `typecheck.sh`, the SBOM and secret-scan gates |
| `docs/` | the mkdocs site, including `docs/adr/` — the Decision Log |
| `stacks/`, `matrix/`, `examples/` | shipped pipeline configs, bake-off matrices, example configs — all executed by gates, none of them decoration |
| `conftest.py` (root) | the determinism guard. Read it before you run anything |

---

## Setup

```bash
uv sync                                          # the CPU-first dev env: core + the CPU reference set
uv run latence-demo                              # end-to-end smoke: CPU-only, offline, exits non-zero on drift
```

The heavy and endpoint Provider packages (the GLiNER family, the `*-endpoint` packages, the LightOn
OCR parsers, `latence-runner-airflow`, `latence-disambig-glinker`) are workspace members but are
**deliberately not installed** into the dev env — they pull torch/transformers, an inference client,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddickmann/latence](https://github.com/ddickmann/latence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
