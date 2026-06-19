---
trigger: always_on
description: >
---


# Nonprofit Data Strategy Skill

This skill guides user and Claude through a structured, interactive process: research the real org → map what they actually do → identify and search for relevant datasets (both top-down and bottom-up) → build a gap analysis → draft and optionally submit formal data requests → build the artifacts the user actually wants.

**The skill is a conversation, not a batch job.** After every phase, stop and share what you've produced, ask for feedback, and wait for the user to confirm before moving on. Do not proceed to the next phase without an explicit green light.

The skill has **two modes**:
- **Quick mode** (15-20 min): Phases 0–2 only. Good for workshops or an initial conversation. Produces a grounded picture of the org's work and a preliminary dataset map.
- **Deep mode** (1-2 hours): All phases. Includes live dataset searching, bottom-up portal discovery, gap analysis, data request drafting and submission, and artifact creation.

---

## Upfront: Orient, Then Ask Two Questions

Start by giving the user a brief overview of what this skill does and how it works — don't just jump into questions. Something like:

*"This skill walks through a structured process for building a data strategy for your nonprofit using NYC open data and other public sources. Here's the full arc:*

| Phase | What happens |
|-------|-------------|
| **0 — Research** | I research your org (website, 990, recent news) before we discuss anything, so the analysis is grounded in what you actually do |
| **1 — Operational map** | We identify your org's branches of work and the data question embedded in each — then you choose which areas to focus on |
| **2 — Dataset brainstorm** | For each area, I surface what datasets would be useful, whether they're public, whether they're on the NYC Open Data portal, and how feasible it would be to request anything that isn't |
| **3 — Live search + bottom-up discovery** | I actually search for each dataset, verify it exists, and browse the portal for datasets we didn't think to look for |
| **3.5 — Data quality audit** | I flag quality issues across the datasets found: gaps, staleness, methodological bias, proxy limitations |
| **4 — Gap analysis** | For each area: what can we build today, and what data is missing and should be sourced |
| **4.5 — Data requests** | I check whether missing datasets have already been requested from the NYC Open Data portal, draft new requests, and can submit them via the portal on your behalf |
| **5 — Artifact drafting** | I build the actual tools — targeting scorecards, advocacy maps, briefs, funder materials — saved as real files |
| **5.5 — Credibility review** | For each artifact, a memo surfacing where data quality issues or missing datasets could affect the conclusions |

*The full process takes 1-2 hours. A quick version (phases 0–2 only) takes about 15 minutes and gives you the operational map and dataset landscape — you can always continue from there."*

Then ask two questions:

**1. Which mode?**
Quick (phases 0–2 only, ~15 min) or deep (all phases, ~1-2 hours)?

**2. Is the Claude in Chrome browser extension set up?**
Several steps in this skill work much better with browser access — specifically:
- Actually retrieving datasets from the NYC Open Data portal to verify they exist and check their fields
- Discovering datasets by browsing the portal directly
- Submitting formal data requests via the NYC Open Data engage form

If the user isn't sure or says no, say: *"You can enable it in Claude settings → Desktop app → Computer use. If you'd rather skip it, that's fine — I'll note where functionality will be limited."* If they decline, proceed anyway and flag those steps as browser-dependent.

---

## Phase 0: Research the Organization

Before asking the user anything about their work, research the org yourself.

**Search for**:
- Their website (homepage, "about," "our work," "programs")
- Their most recent Form 990 via ProPublica Nonprofit Explorer (projects.propublica.org/nonprofits) — this often reveals programs and scale that aren't on the website
- Any recent news, reports, or publications

**Present your findings** in a short summary covering:
- Mission (quote their own language where possible)
- Main programs and branches of work — what they *actually do*, not just what category they fit
- Rough scale: budget, staff, geography served, populations served
- Any mentions of data, research, or metrics they already track

Then stop: *"Here's what I found about [org name]. Does this accurately reflect your work? Anything I've missed or gotten wrong?"*

Wait for confirmation before proceeding.

**Why this step matters**: Skipping it leads to generic operational maps that don't reflect how the org actually works. A housing org that only does legal services has completely different data needs than one that also does community organizing or manages affordable housing units. The 990 in particular reveals programs invisible on the website.

---

## Phase 1: Map Operational Areas, Then Scope

Based on your research (not generic categories), draft a specific map of what this org actually does. Use the org's own language for their programs and activities.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JordanSucher/open-data-nonprofit-assistant](https://github.com/JordanSucher/open-data-nonprofit-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
