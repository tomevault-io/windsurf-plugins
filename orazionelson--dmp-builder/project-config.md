---
trigger: always_on
description: Build Data Management Plans for research projects, scientific institutions (universities, research centres, ERIC infrastructures, libraries with research data) and FAIR-aligned contexts. Activate this skill when the user asks to write a DMP for funded projects (Horizon Europe, ERC, MSCA, PRIN/MUR, DFG, ANR, NWO, SNSF, AEI, FCT, NCN, other European funders), for research or academic institutions, for research infrastructures, for Open Science or FAIR data projects, or for institutional repositori
---


# DMP Builder — research, institutions, FAIR

Skill for building Data Management Plans across three context families:

1. **Funded projects** (Horizon Europe, ERC, MSCA, PRIN/MUR, DFG, ANR, NWO, SNSF, AEI, etc.)
2. **Research institutions** (universities, centres, ERIC infrastructures, digital libraries)
3. **FAIR-aligned contexts** without a specific funder

> For the **Italian public administration** (municipalities, ASL local health authorities, public schools with citizen-facing services) there is a dedicated skill: `governance-dati-pa`. That domain is regulated by CAD, Italian GDPR, Legislative Decrees 33/2013, 36/2006, 144/2024, with effects quite different from research. If the use case is PA, redirect.

---

## Orientating principle: European digital sovereignty

When the DMP requires third-party services (data repositories, DOI provider, storage, compute, code hosting, communication, persistent identifiers, bibliographic management), this skill primarily recommends:

1. **Services with European incorporation**: Zenodo (CERN), EUDAT B2SHARE/B2HANDLE/B2DROP, OpenAIRE, DataCite (Hannover-based), EOSC, OVHcloud (FR), Hetzner (DE), Scaleway (FR), Aruba Cloud (IT), IONOS (DE), Open Telekom Cloud (DE), Codeberg (DE, e.V.), Framagit (FR, Framasoft), Nextcloud (DE), CryptPad (FR, XWiki SAS), Element/Matrix (UK-FR), BigBlueButton (open source, EU-hostable), Jitsi (open source, EU-hostable), HedgeDoc (open source, DE-NL community).
2. **As alternative, open source self-hostable on EU infrastructure**: Forgejo/Gitea, Harbor, GitLab CE/EE, Galaxy Europe, institutional Dataverse, Frictionless Data, FAIR Data Point.

US-incorporated proprietary services (GitHub.com, AWS, Azure, GCP, Google Drive, Dropbox, Figshare, Mendeley, Slack, Notion, Zoom, Microsoft Teams) are acceptable **only** when:

- The institution mandates them as internal standard with which the project must align
- No equivalent alternative exists for the specific case (e.g. ORCID remains de facto, no European equivalent exists)
- The data passing through them is already public/non-sensitive (e.g. GitHub only for open source code without restricted data)

In any case, **justify the choice in the DMP** and indicate a migration plan. See `references/european-services.md` for the full per-category map.

> Important note on **ORCID** and **ROR**: these are de facto international standards for researcher and organisation identity. Both are incorporated as non-profits in the United States. No European alternative has equivalent adoption. The skill recommends them regardless, because their absence would damage *findability* and *interoperability* of the data — the very FAIR principles the DMP intends to guarantee. Document in the DMP that the choice is driven by standard pervasiveness.

---

## Operational flow

### 1. Understanding the context (eliciting)

Three questions, skip those already answered:

1. **Context**: funded project (which funder?) / institutional / FAIR-aligned without funder
2. **Predominant data type**: experimental quantitative / observational / qualitative / computational (simulations) / synthesis (review/meta) / geospatial / biological sequences / digital cultural heritage / mixed
3. **Expected deliverable**: full plan / template to fill / checklist / FAIR self-assessment / specific section (e.g. publication strategy only)

If the user explicitly invites a collaborative session (*"let's write it together", "step by step"*), switch to §1a.

### 1a. Collaborative mode (iterative co-authoring)

**Trigger**: "let's write it together", "let's work on it", "iteratively", "step by step", "block by block", or when the user provides specific project input (acronym, funding, WPs, expected datasets) to incorporate.

**Behaviour**: in collaborative mode **do not dump the full template as the first reply**. Work in blocks:

1. **Project identification**: title/acronym, funding, partners, PI, ORCID, duration. 2–3 questions.
2. **Inventory of planned datasets**: one card per dataset (name, type, estimated volume, project phase, reuse of existing data).
3. **For each dataset**: metadata standards, quality control, active storage, sharing strategy, licence, target repository, PID strategy.
4. **Cross-cutting sections**: legal/ethical (GDPR, consents, IP, ethics committee), resources and responsibilities (data steward, costs).
5. **FAIR self-assessment**: a status table for each of the four dimensions.
6. **Final output**: the complete DMP as a file, tailored to the case.

At each block: 1–3 targeted questions, summary, explicit confirmation before proceeding.

### 2. Composing the deliverable

**Asset = deliverable** (call `present_files`). **Reference** = informs the response, not presented.

| User request | Primary asset (present) | References to consult |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orazionelson/dmp-builder](https://github.com/orazionelson/dmp-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
