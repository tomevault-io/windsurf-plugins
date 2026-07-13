---
trigger: always_on
description: Operating protocol for AI coding agents working in this repository.
---

# AGENTS.md

Operating protocol for AI coding agents working in this repository.

Claude Code should read `CLAUDE.md` first, then use this file as the shared repository contract. Other agents should start here. Follow repository-local guidance over generic awesome-list assumptions.

## Repository North Star

This is a public, maintained field guide for securing agentic, multi-agent, tool-using, memory-bearing, and cyber-capable AI systems. The `README.md` is the entry point, but the repository is more than a link index: it pairs a curated catalogue with conceptual maps (`docs/`), secure engineering patterns (`patterns/`), evaluation rubrics (`rubrics/`), reference visuals (`visuals/`), and first-party executable assets (`agents/`, `skills/`, `hooks/`).

Everything is organised around the AI Defense Plane: **Discover** where agents, tools, prompts, data flows, credentials, memory, and autonomous workflows exist; **Protect** tool use, memory writes, credentials, and actions; and **Govern** evidence, audit trails, delegated authority, and risk acceptance.

The list is curated, not accumulated. Each entry should help a reader understand an agentic security risk, find a credible control or test case, or compare related resources. Selectivity, durability, clear placement, and neutral, evidence-led description quality matter more than volume.

## Agent Role

Agents may help with:

* README and catalogue maintenance when explicitly asked
* New entry review against the quality bar
* Pull request review
* Issue triage using the repository templates
* Broken-link checks
* Duplicate detection
* Section placement within the existing taxonomy
* Description tightening and de-hyping
* Scoresheet checks for new resources, benchmarks, and case studies
* Maintainer comment drafts
* Small, safe cleanup edits when explicitly requested

Agents must not:

* Add speculative, promotional, or low-signal entries
* Inflate claims or preserve marketing wording
* Include operational exploit detail beyond what defensive understanding requires
* Reorganise the taxonomy without explicit instruction
* Run broad formatting or link-rewriting sweeps
* Edit unrelated files
* Rewrite the maintainer's style unnecessarily
* Turn one contribution into a broad structural change
* Commit generated, local, or protected material (see Protected Areas)
* Touch protected areas unless explicitly instructed

## Read Order

Before reviewing or editing, read in this order:

1. `README.md` — scope, taxonomy, section formatting, protected blocks, and existing examples
2. `CONTRIBUTING.md` — useful contributions, evidence requirements, and editorial standards
3. `rubrics/README.md` — the scoresheet requirement for new resources, benchmarks, and case studies
4. `.github/ISSUE_TEMPLATE/` — contributor expectations for resources, benchmarks, case studies, and pattern improvements
5. `.github/pull_request_template.md` — contribution types, evidence fields, and the PR checklist
6. `CLAUDE.md` — coding standards and the Claude-specific review format
7. Recent issues and merged PRs, where available, for maintainer precedent

Do not assume the generic awesome-list pattern overrides this repository's structure.

## Repository Facts

* `README.md` contains introductory sections, a Contents list with anchors, the AI Defense Plane framing, and themed list sections such as Standards and Frameworks, Prompt Injection, Tool Use / MCP / Runtime, Memory and State, Credentials and Authority, Benchmarks, Cyber-Capable AI Agents, Observability, Governance, and Engineering Patterns.
* List sections use a mixture of bullets and tables. Match the local section style exactly.
* Many sections include explanatory text before entries. Preserve it.
* The catalogue is split between `README.md` (curated highlights) and the fuller indexes in `resources/` (`standards-and-frameworks.md`, `tools.md`, `benchmarks.md`, `papers.md`, `vendor-research.md`, `cyber-capable-ai-agents.md`). Add detailed metadata entries to the relevant `resources/` file; add only high-signal highlights to `README.md`.
* `CONTRIBUTING.md` asks for narrow, reviewable changes and one suggestion per pull request.
* New resources, case studies, and benchmarks require a corresponding scoresheet under `rubrics/scoresheets/`, using the rubrics in `rubrics/`.
* New entries should be added to the bottom of the relevant section unless local ordering clearly indicates otherwise.
* New sections or taxonomy changes should be handled separately from single-entry contributions.
* For tools and frameworks, prefer the official GitHub repository over a package registry or marketing page.
* For papers, prefer the official publisher page, arXiv, or DOI.
* Descriptions are short, neutral, present-tense where possible, and evidence-led.


## Scope Rules

Belongs:

* Standards, frameworks, and public specifications (for example OWASP, NIST, MITRE ATLAS)
* Vendor research and independent research with clear relevance to agentic AI security
* Papers, technical reports, and durable explainers
* Datasets and benchmarks that test agentic behaviour, tool use, memory, credentials, autonomy, or multi-agent workflows
* Evaluation, red-teaming, observability, tracing, audit, and forensics tooling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [natnew/Awesome-Agentic-AI-Security](https://github.com/natnew/Awesome-Agentic-AI-Security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
