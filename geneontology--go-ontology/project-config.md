---
trigger: always_on
description: This includes instructions for editing the go ontology and related files in response to GitHub issues.
---

# GO Ontology Project Guide

This includes instructions for editing the go ontology and related files in response to GitHub issues.

## Project Layout

The project follows standard ODK layout:

 * src
    * ontology/
       * go-edit.obo
       * Makefile
       * ...
 * terms/   ## checked out copies, for editing
    * GO_NNNNNNN.obo # checked out local copy
 * .claude/
    * skills/   ## specific skill instructions

 For make targets, a standard pattern is `cd src/ontology && make <TARGET>`. Beware of accidentally changing to the `src/ontology`
 dir and forgetting where you are. It is best to run things from the top level

These instructions are optimized for claude code. Skills are used, and defined
in `.claude/skills/`

## Local Setup (skip if running in a GitHub Action)

The editing workflow below relies on two sets of tools:

1. **ODK-provided tools** — `robot`, `owltools`, `dosdp-tools`, etc. These come from the pinned `obolibrary/odkfull` Docker image and **must not** be installed from Homebrew, the upstream releases page, or any other source. CI runs against the pinned image, and any other copy will drift in version and may give results that don't match CI. See the AUTOMATED-VALIDATION section and the `/odk-make` skill below for how to invoke them.
2. **obo-scripts** — three Perl scripts (`obo-grep.pl`, `obo-checkout.pl`, `obo-checkin.pl`) used for stanza-aware search and the checkout/checkin procedure. These are not part of ODK and have to be on `PATH` separately.

When this repo is driven by the `ai-agent` / `copilot-setup-steps` workflows, both are taken care of (the job runs inside the ODK container, and obo-scripts is cloned in) — skip this section.

For a local editor-driven Claude Code session:

**ODK tools.** Install Docker if you do not already have it. Do not install `robot` or other ODK tools on the host. Drive every `make` / `robot` / `owltools` / `dosdp-tools` invocation through the `/odk-make` skill's non-interactive wrapper, e.g.

```bash
.claude/skills/odk-make/odk-run.sh robot --version
.claude/skills/odk-make/odk-run.sh make travis_build
```

The wrapper uses the same image tag as `src/ontology/run.sh` (the canonical console invocation), so the version of `robot` you run matches the one CI uses. See the `/odk-make` skill for details and common targets.

**obo-scripts.** Check whether they are on `PATH`:

```bash
which obo-grep.pl obo-checkout.pl obo-checkin.pl
```

If any are missing, install from https://github.com/cmungall/obo-scripts:

```bash
mkdir -p ~/.local/share
git clone --depth 1 https://github.com/cmungall/obo-scripts.git ~/.local/share/obo-scripts

mkdir -p ~/bin
ln -sf ~/.local/share/obo-scripts/obo-grep.pl     ~/bin/obo-grep.pl
ln -sf ~/.local/share/obo-scripts/obo-checkout.pl ~/bin/obo-checkout.pl
ln -sf ~/.local/share/obo-scripts/obo-checkin.pl  ~/bin/obo-checkin.pl
```

If `~/bin` is not already on `PATH`, add it (`export PATH="$HOME/bin:$PATH"` in your shell rc) or substitute a directory that is. The `editing-obo-ontologies` skill, if available, also bundles these.

Do not fall back to plain `grep` or to hand-editing `go-edit.obo` — that path is much slower and error-prone, and the rest of this guide assumes the tools are in place.

## PLAN: Analyze Issue, Plan Approach, and create a TODO/checklist

Read the entire issue and all associated comments. Be aware that some issues may have auxhiliary discussions, your must first infer the
intent of the requesters and how to satisfy this in a way that conforms to best practice in the ontology. If the user intent from the
issue is hopelessly ambiguous, then write a note asking for clarification in the original issue (using `gh`) and end here.

Create a plan for addressing the issue. The plan MUST have the following components (you can have more components depending on the nature of the issue)

- [ ] PLAN: The issue and all its context has been analyzed, the intent is clear, and a plan for addressing it has been created
- [ ] PRE-VALIDATION: Current state of the ontology validates prior to any changes (if not, we can't validate our changes)
- [ ] RESEARCH: If appropriate, necessary background research performed; always use /research for this, it produces a RESEARCH.md file
- [ ] TERM-SEARCH: Relevant ontology terms (this ontology or others) have been consulted
- [ ] DESIGN-PATTERNS: Existing design patterns, terms, and documentation consulted; always use /design-pattern for this, which produces a DESIGN_PATTERNS.md doc
- [ ] EDITS: correct procedure is followed for making edits, using checkin/checkout commands, and local `./terms/ folder
- [ ] RELATIONSHIPS: appropriate relationships and logical axioms are included
    - [ ] logical definitions are appropriate, and are not over-specified, and conform to DESIGN_PATTERNS.md doc
    - [ ] relationships conform to other similar terms in the ontology
    - [ ] is_a is not over-asserted
    - [ ] is_a, part_of, and other relationships specified appropriately
- [ ] SPECIALIZED-EDITS: include checklists from the following skills as appropriate, depending on the nature of the request
    - /term-obsoletion skill; MUST be used for anything involving obsoletion/deprecation of terms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geneontology/go-ontology](https://github.com/geneontology/go-ontology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
