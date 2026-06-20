---
trigger: always_on
description: Find, verify, cite, visualize, and route TCIA-published datasets and verified manuscripts about TCIA data across TCIA WordPress Collection and Analysis Result metadata, TCIA Publications EndNote XML, IDC/idc-index, Cancer Data Aggregator, General Commons, CTDC, PathDB, DataCite, and Aspera. Use when users ask to discover TCIA datasets or publications by cancer type, modality, body site, species, data type, access/license, DOI, program, clinical/supporting data, demographic/diagnosis enrichment, 
---


# TCIA Query Skill

## Core Rule

Use the TCIA WordPress Collection Manager as the authority for whether a dataset is TCIA-published. A dataset is in scope only if it appears as a WordPress Collection or Analysis Result. For normal agent work, query the local SQLite snapshot as the fast discovery surface over WordPress, PathDB, and DataCite metadata. Downstream systems such as IDC, CDA, General Commons, CTDC, PathDB, Zenodo, and DataCite can enrich or route access, but they do not decide TCIA provenance.

Use TCIA's Publications page and EndNote export as the authority for peer-reviewed manuscripts written about TCIA datasets: `https://www.cancerimagingarchive.net/publications/` and `https://cancerimagingarchive.net/endnote/Pubs_basedon_TCIA.xml`. DataCite records describe TCIA dataset DOI metadata; they are not the verified bibliography of papers that used TCIA data. For questions about papers, manuscripts, publication impact, hypotheses studied, methods, or citation lists, load `references/publications.md` and use `scripts/tcia_publications.py`.

Exception for DOI-centered questions: start with DataCite for DOI metadata, citation metadata, versions, and DOI relationships, then use WordPress to confirm TCIA publication status, visibility, access/license, and user-facing dataset pages.

Ignore WordPress records where `hide_from_browse_table = "1"` by default. TCIA uses this flag for pre-release staging/review and for retired/outdated datasets that should not be casually rediscovered. Include hidden records only when the user explicitly says they are a TCIA staff member and explicitly asks to include hidden, staged, retired, or internal-review datasets.

When a downstream record is derived from a TCIA DOI but is not itself listed in WordPress, describe it as an external derived or related dataset, not as a TCIA-published dataset.

## Quick Workflow

0. Use the local SQLite metadata snapshot for routine discovery and access/license metadata. Prefer the agent-facing views in `references/schema.md`: `agent_datasets`, `agent_current_downloads`, `agent_dataset_access_summary`, `agent_pathdb_slides`, and `agent_datacite_dois`. Load `references/snapshots.md` for refresh behavior and `references/schema.md` for SQL details. If the environment cannot execute scripts or query SQLite, use the web-friendly release exports described in `references/mcp-and-web-llms.md`; do not switch to live WordPress API discovery.
1. Choose the starting source.
   - For peer-reviewed publications or manuscripts about TCIA data, use TCIA's EndNote XML export, not DataCite. Prefer `scripts/tcia_publications.py`, and load `references/publications.md`.
   - For DOI, citation, or version questions, start with DataCite metadata from the snapshot. Prefer `scripts/datacite_tcia_dois.py` for TCIA DOI prefix records.
   - For subject-level clinical, demographic, diagnosis, treatment, or cross-commons data-availability enrichment, confirm the TCIA dataset in WordPress first, then use CDA from validated TCIA/IDC subject identifiers. Load `references/cda.md`.
   - For file-level public NIfTI questions, confirm TCIA provenance/access through the normal snapshot first, then load `references/nifti.md` and use the optional NIfTI SQLite release only if file-grain metadata are needed. Prefer `agent_nifti_dataset_summary`, `agent_nifti_downloads`, `agent_nifti_files`, and `agent_nifti_derived_objects`.
   - For controlled-access file, manifest, `drs_uri`, modality, or series-level metadata, confirm controlled status through the normal snapshot first, then load `references/controlled-access.md` and use the optional controlled-access SQLite release when file-grain public metadata are needed. Prefer `agent_controlled_dataset_summary`, `agent_controlled_downloads`, and `agent_controlled_files`.
   - For all other discovery and access questions, search WordPress snapshot records first for Collections and Analysis Results.
   - Prefer `scripts/tcia_wordpress_search.py` for lightweight snapshot searches.
   - If the snapshot is missing, run or tell the user to run `python scripts/tcia_snapshot.py ensure` from the skill root.
   - If a specific dataset appears absent after refreshing, say the snapshot may not include the newest TCIA metadata yet. The GitHub Action builds release snapshots at 7:17 AM and 7:17 PM America/New_York; ask the user to try again after the next scheduled run has had time to finish, then rerun `python scripts/tcia_snapshot.py ensure`.
   - Use `--include-hidden` only for explicit TCIA staff requests that ask for hidden/staged/retired records.
2. Filter out `hide_from_browse_table = "1"` records unless the explicit TCIA staff exception applies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirbyju/tcia-query-skill](https://github.com/kirbyju/tcia-query-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
