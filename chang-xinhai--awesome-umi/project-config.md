---
trigger: always_on
description: This repository is a curated **Awesome-UMI** list for the ecosystem around **Universal Manipulation Interface (UMI)**.
---

# AGENTS.md

## Mission

This repository is a curated **Awesome-UMI** list for the ecosystem around **Universal Manipulation Interface (UMI)**.

The goal is not to be a generic robot-manipulation bibliography. The goal is to organize the most relevant resources for:
- UMI core papers and official resources
- UMI-style methods for robot-free teaching, teleoperation, and cross-embodiment transfer
- UMI-related datasets and dataset families
- policies trained on UMI data or commonly used as UMI-relevant baselines / support models
- structured taxonomy knowledge such as embodiments, modalities, data formats, deployment constraints, and surveys

## Single-file content policy

All curated content lives in `README.md`.

Do **not** create `contents/` pages for paper organization.
Do **not** split the awesome list into multiple markdown content files unless the user explicitly requests it.

Allowed root-level files:
- `README.md` — the canonical awesome list
- `AGENTS.md` — governance and maintenance rules
- `LICENSE`
- optional static assets under `assets/` or `imgs/` when they improve presentation

## Canonical section layout

`README.md` should use this top-level structure:
1. `UMI Core`
2. `UMI Method`
3. `UMI Dataset`
4. `UMI Policy`
5. `UMI Taxonomy`

Use a Table of Contents near the top of the README and keep subsection anchors stable.

## Section definitions

### UMI Core
Include only the foundational layer:
- the original UMI paper
- official project / code / dataset community site
- direct follow-up papers that define the UMI ecosystem
- clearly UMI-centered survey / positioning papers

### UMI Method
Include methods strongly relevant to UMI-style collection, transfer, and deployment.
Typical subsections include:
- Human Demonstration Interfaces
- Teleoperation / Robot-Free Teaching
- Cross-Embodiment Transfer
- Bimanual Manipulation
- Dexterous Hand / DexHand
- Multimodal / Force / Tactile Extensions
- In-the-Wild / Egocentric / Mobile Extensions

Do **not** expand this section into a general manipulation-method survey.

### UMI Dataset
Include all relevant datasets for the UMI ecosystem.
This includes:
- official UMI datasets
- direct UMI-native dataset families from the community site
- strongly related dexterous / bimanual / multimodal / mobile dataset families
- method papers that also release a dataset

### UMI Policy
Include policies that are:
- trained on UMI data,
- evaluated on UMI-like datasets/tasks,
- or commonly used as relevant policy baselines / support models for the ecosystem.

Typical subsections:
- Imitation Learning
- Diffusion / ACT-style Policies
- VLA
- Cross-Embodiment Policies
- Bimanual Policies
- Dexterous Policies
- Deployment / Real-World Transfer

Do **not** include every famous robot policy by default.

### UMI Taxonomy
This is the structured-knowledge section.
It may contain papers, websites, docs, conventions, or notes.
Typical subsections:
- Embodiments
- Observation Modalities
- Action Spaces
- Data Formats / Storage Conventions
- Deployment / System Constraints
- Benchmarks / Evaluation Dimensions
- Survey Mentions / Taxonomic Discussions
- Best Practices / Standards / Notes

## Inclusion policy

The repository uses **core + strong adjacent** scope.

### Include first
- direct UMI papers and official resources
- projects and datasets listed on `https://umi-data.github.io/`
- direct descendants and strong neighboring work around robot-free teaching, teleoperation, cross-embodiment transfer, dexterous UMI, multimodal UMI, and UMI-style deployment

### Include selectively
- policy papers such as ACT, Diffusion Policy, or VLA systems when they are clearly used by or relevant to UMI-style training and evaluation
- survey / taxonomy papers that explicitly help explain the UMI ecosystem

### Usually exclude
- generic manipulation papers with weak or no UMI connection
- broad teleoperation or imitation-learning papers that do not materially help understand or use UMI
- unrelated robot planning, control, or benchmark papers

## Source priority

When adding or updating entries, use sources in this order:
1. official UMI paper / official project / official code
2. `https://umi-data.github.io/`
3. Google Scholar citation graph around the UMI paper
4. official project pages and GitHub repos for descendants
5. arXiv / conference pages for canonical paper metadata

If multiple sources disagree, prefer the most official public source.

## Verification rule for new entries

Do **not** add an entry unless it has a sufficiently reliable canonical public source.

Minimum bar for inclusion:
- a canonical paper URL such as arXiv / conference page / journal page,
- and/or an official project page, dataset page, or GitHub repo from the authors / lab / organization

If a candidate cannot be verified with enough confidence:
- do **not** add it to `README.md`
- do **not** guess missing metadata, URLs, venues, or affiliations
- do **not** include partially confirmed or rumor-like items just because they seem relevant

When in doubt, exclude first and wait for a reliable source.

## Entry format

Each subsection should use the same compact table schema:

| Date | Keywords | Institute (first) | Paper | Publication | Others |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chang-xinhai/Awesome-UMI](https://github.com/chang-xinhai/Awesome-UMI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
