---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Codex CLI, Cursor, Copilot) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex CLI, Cursor, Copilot) working in this repository.

Related docs: [docs/CONTRIBUTING.md](docs/CONTRIBUTING.md) (contributor workflow), [docs/development.md](docs/development.md) (validation commands), [docs/INSTALL.md](docs/INSTALL.md) (installation).

---

## Repository overview

Agents and skills covering bioinformatics, literature discovery, scientific writing, and data visualization. Runs under Claude Code and the Codex CLI. `catalog/catalog.json` is the countable source of truth for how many of each exist; do not restate the counts in prose.

Layout:
- `agents/` — agent definitions (markdown)
- `skills/` — skill directories; each has a `SKILL.md`
- `scripts/` — router, catalog builder, hook installer, validators, benchmark
- `tests/` — unit tests + routing benchmark
- `catalog/` — generated routing artifact (`catalog.json`)
- `docs/` — MkDocs site sources, routing model, benchmark baseline
- `Makefile` — the installer: install, catalog, hook, benchmark, uninstall, status, validate

Install: `make install` symlinks agents and skills into `~/.claude/` and `~/.codex/`.

---

## Default Workflow Selection

When working in this repository without a specialized agent prompt, do not choose skills ad hoc.

Start with the catalog:

```bash
python3 scripts/skill_index.py route "<task>"
```

Use the returned agent, primary skills, and suggested order as the default workflow. Then open the referenced agent file in `agents/` and the referenced `SKILL.md` files before proceeding. Only deviate from the returned path when the request clearly falls outside the suggested workflow.

For installed environments outside the repository checkout, use:

```bash
python3 ~/.agents/omics-skills/skill_index.py route "<task>"
```

---

## Scientific Workflow Guardrails

For omics or scientific project work, agents must maintain an explicit reasoning loop instead of treating analysis steps as a linear pipeline.

1. **Hypothesis register**: before the first analysis step, create at least 5 distinct working hypotheses or explanations. Include technical artifacts and null explanations alongside biological mechanisms. If fewer than 5 are plausible, state why and add discriminating negative controls or failure modes.
2. **Intermediate reflection**: after each major intermediate result or QC gate, write a short reflection covering what was observed, whether the result passed QC, which hypotheses gained or lost support, what alternative explanations remain, and the next discriminating check.
3. **Literature context**: after the initial hypothesis register and after any unexpected, central, or final finding, run an additional literature search using `polars-dovmed` or another appropriate literature-search skill. Use broad synonym-aware queries, summarize the relevant evidence with DOI/PMCID when available, and state whether the literature supports, contradicts, or narrows each leading hypothesis.
4. **Hypothesis revision**: revise and rank the hypothesis register as evidence accumulates. Do not silently discard hypotheses; mark them as supported, weakened, ruled out, or unresolved with the evidence that changed the status. Keep at least 5 active hypotheses while the project is exploratory; generate replacements when hypotheses are ruled out.
5. **Final synthesis**: final reports must include hypotheses considered, intermediate reflections, literature context, revised hypothesis ranking, and the next experiments or analyses that would best separate remaining alternatives.

These guardrails apply to agent prompts, skill instructions, examples, and documentation added or edited in this repository.

---

## Literature-Derived Discovery Guardrails

Omics agents must actively ask what is biologically interesting in the data. Do not stop at "workflow completed", "annotation complete", or "QC passed" when the user has provided genomes, contigs, MAGs, viral genomes, proteins, or annotations.

1. **Infer the biological context first**: use the available taxonomy, marker genes, genome statistics, sequence similarity, and QC evidence to infer the likely organism, virus group, sample context, and closest plausible references.
2. **Build a literature-derived analysis playbook**: before deciding what is "interesting", search the relevant literature for that inferred group and summarize what scientists typically analyze, which comparison sets they use, which features/outliers they report, and which tools or markers are considered appropriate. Prefer review papers, recent primary studies, and benchmark/tool papers for the same clade or data type.
3. **Choose methods from that playbook**: select comparative, phylogenetic, annotation, structural, statistical, and visualization analyses because they match the literature and the data, not because a fixed global checklist says so. Document why each chosen analysis is appropriate and which plausible analyses were skipped.
4. **Compare like with like**: identify close relatives or relevant reference sets using literature-supported methods for the inferred group. For example, do not apply a phage-oriented clustering workflow to NCLDV-style giant viruses unless the literature supports it for that case.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fmschulz/omics-skills](https://github.com/fmschulz/omics-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
