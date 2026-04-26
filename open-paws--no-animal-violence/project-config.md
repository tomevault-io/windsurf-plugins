---
trigger: always_on
description: Rule files for detecting speciesist language in code, documentation, and prose. Part of the Open Paws speciesist language detection suite. Compatible with woke, alex/retext-equality, and Vale.
---

# no-animal-violence

Rule files for detecting speciesist language in code, documentation, and prose. Part of the Open Paws speciesist language detection suite. Compatible with woke, alex/retext-equality, and Vale.

## Quick Start

Rules are static YAML/config files — no build step required. Copy or reference the relevant directory for your tool:

- **woke**: use `woke/.woke.yaml`
- **alex/retext-equality**: use files in `alex/`
- **Vale**: use the `vale/Speciesism/` style package

## Architecture

This is a **mono-repo of rule definitions** for three different inclusive-language scanners. Each tool has its own directory with rules in its native format. All rule sets detect the same categories: violent animal idioms, animal-as-object metaphors, and speciesist technical terminology.

## Key Files

| File | Description |
|------|-------------|
| `woke/.woke.yaml` | woke scanner config with all speciesist patterns |
| `alex/animal-violence.yml` | alex/retext-equality rules for violent idioms |
| `alex/speciesism.yml` | alex/retext-equality rules for speciesist metaphors |
| `alex/industry-euphemisms.yml` | alex/retext-equality rules for industry euphemisms (free-range, cage-free, harvest) |
| `vale/Speciesism/AnimalIdioms.yml` | Vale rule: violent animal idioms |
| `vale/Speciesism/AnimalMetaphors.yml` | Vale rule: animal-as-object metaphors |
| `vale/Speciesism/TechTerminology.yml` | Vale rule: speciesist tech terms |
| `vale/Speciesism/meta.json` | Vale style package metadata |

## Organizational Context

This repo is the **canonical rule dictionary** for the entire no-animal-violence suite. Every downstream adapter (Vale, ESLint, Semgrep, VS Code, pre-commit, GitHub Action, reviewdog, Danger) consumes or mirrors these definitions. Rule additions here should propagate to all consumers.

**Strategic significance (Lever 1 + Lever 3):** This is a concrete Lever 3 tool — it embeds animal welfare standards into the development process itself. Every developer who installs an adapter becomes aware of speciesist language patterns. The suite is intervention #8 from the 25 Ranked Interventions (Animal-Friendly Data Labelling Standards) made concrete.

**Academic backing:** Rules are backed by peer-reviewed research on AI speciesism and language bias. Key citations:
- Hagendorff et al. (2023–2025) — speciesist bias in LLM outputs; doi:10.1007/s43681-023-00380-w
- Takeshita et al. (2022) — language normalization of animal exploitation
- Leach et al. (2023) — moral circle expansion and language framing

These citations appear in rule metadata (`references:` field) and give the suite credibility in Lever 3 outreach to development teams and AI labs.

**MCP ecosystem integrations:**
- `lbr8-mcp-constraints` — bundles 12 offline NAV patterns sourced from this repo as a `StaticConstraintSource`. These patterns run in air-gapped/offline contexts where the full Semgrep ruleset cannot be loaded. When updating rules here, check whether the 12 bundled patterns in `lbr8-mcp-constraints` need updating. See `repos/lbr8-mcp-constraints` in the strategy repo.
- `mcp-server-nav-language` — loads `semgrep-rules-no-animal-violence` YAML at startup for the content pipeline NAV gate (PR #7). Canonical patterns flow: this repo → `project-compassionate-code` generator → `semgrep-rules-no-animal-violence` → `mcp-server-nav-language`.

**Current org priorities relevant to this repo:**
- The suite has zero platform presence despite being a shipped product. Planned: bootcamp setup instructions (VS Code extension + pre-commit hook), guild developer onboarding, platform CI. See `ecosystem/integration-todos.md` §27a.
- Suite maintenance has **no named owner** as of 2026-04-02. The 65+ rules need ongoing false-positive tuning and new term additions — ideal F/E-rank quests in the Guild pipeline.
- Adoption metrics (npm downloads, VS Code marketplace installs, GitHub Action usage) are not yet tracked. This is a Lever 3 measurement gap.
- Compassionate Code integration is in place — the CC scanner uses these rules to find speciesist language in open-source repos. Stronger rules here directly improve CC scanner output.

**Decisions affecting this repo:**
- 2026-03-25: Every repo in the org runs `semgrep --config semgrep-no-animal-violence.yaml` on all code/docs edits as a quality gate.
- 2026-04-01: The no-animal-violence suite is the model for how shared canonical configs should work org-wide (the external contribution safety pattern should mirror this structure).

## Related Repos

- [vale-no-animal-violence](https://github.com/Open-Paws/vale-no-animal-violence) — Standalone Vale distribution package
- [eslint-plugin-no-animal-violence](https://github.com/Open-Paws/eslint-plugin-no-animal-violence) — ESLint plugin for JS/TS
- [semgrep-rules-no-animal-violence](https://github.com/Open-Paws/semgrep-rules-no-animal-violence) — Semgrep rules
- [vscode-no-animal-violence](https://github.com/Open-Paws/vscode-no-animal-violence) — VS Code extension
- [no-animal-violence-action](https://github.com/Open-Paws/no-animal-violence-action) — GitHub Action

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Open-Paws) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
