---
trigger: always_on
description: This repo is classified as **T4 — Isolated** under the Infrastructure Governance Standard.
---

## Infrastructure Governance
This repo is classified as **T4 — Isolated** under the Infrastructure Governance Standard.
Before modifying shared infrastructure, read: `shared-library/standards/INFRASTRUCTURE-GOVERNANCE.md`
- **Deploy target:** local-only (research)
- **Dependencies:** None
- **Dependents:** None

# CLAUDE.md — Execution Contract
## indeterminate-emergence

## Framework Dependencies
> Canonical specs live in [shared-library](https://github.com/cerberusxops360/shared-library).
> Read these before modifying any framework-level behavior in this repo.

| Dependency | Spec | Why |
|------------|------|-----|
| Voice & Writing Standards | `standards/voice-writing/VOICE-STANDARDS.md` | Paper: MODERATE filter. Blog post: STRICT filter. |
| Epistemic Security | `frameworks/epistemic-security/ES-SPEC.md` | This repo IS the canonical implementation |

---

**Repo:** `cerberusxops360/indeterminate-emergence`  
**Project Type:** Open research — security theory + proof of concept  
**Author:** Adam Bishop, XOps360 LLC  
**ORCID:** https://orcid.org/0009-0000-4569-3726  
**License:** Paper: CC BY 4.0 | Code: MIT  
**ADAM Project ID:** ADAM-PROJ-IE  
**Status:** Phase 1 — Repository Setup & Paper Finalization  

---

## What This Repo Is

A security framework and proof-of-concept implementation for **indeterminate emergence** — a primitive where meaningful information, capabilities, and effects do not exist outside authorized execution contexts. The framework extends deniable encryption concepts to real-time system behavior, using Pufferfish-style privacy definitions and differential privacy composition to provide formal guarantees against capability inference attacks.

This is an open research project. The repo contains:
1. The academic paper (markdown, LaTeX, PDF)
2. A blog post (plain-language explanation)
3. A proof-of-concept proxy service demonstrating AI capability-set inference resistance
4. Evaluation experiments with quantitative results

---

## ADAM Integration

### Object Types

This repo introduces the following ADAM-managed object types:

```
RESEARCH_PAPER
  fingerprint: SHA-256 of paper markdown content
  tier: T1 (warm — actively edited during revision cycles)
  relationships: PRODUCES blog_post, SPECIFIES poc_system
  status: draft | submitted | published | revised

BLOG_POST
  fingerprint: SHA-256 of blog markdown content
  tier: T1 (warm — actively edited during publication cycle)
  relationships: DERIVED_FROM research_paper, PUBLISHED_TO platform_id

POC_COMPONENT
  fingerprint: SHA-256 of source file content
  tier: T1 (warm — actively developed)
  relationships: IMPLEMENTS paper_section, DEPENDS_ON component_id

EVAL_EXPERIMENT
  fingerprint: SHA-256 of experiment script + config
  tier: T1 (warm — actively running)
  relationships: VALIDATES paper_claim, USES poc_component

EVAL_RESULT
  fingerprint: SHA-256 of result data
  tier: T2 (cool — reference data, changes only on re-run)
  relationships: PRODUCED_BY eval_experiment, VALIDATES paper_claim

SUBMISSION_RECORD
  fingerprint: SHA-256 of submission metadata JSON
  tier: T3 (cold — changes only on new submission)
  relationships: SUBMITS research_paper, TARGETS venue_id
```

### Fingerprint Format

Standard ADAM format: `fp-{type}-{YYYYMMDD}-{8hex}-{4hex}`

Type codes for this repo:
```
rpap  — research paper
blog  — blog post
pocc  — proof of concept component
eval  — evaluation experiment
evrs  — evaluation result
subm  — submission record
```

Example: `fp-rpap-20260313-a7b3c9d2-e4f5`

### Manifest Entries

Every versioned artifact gets a manifest entry. Paper versions are tracked by content hash — if the hash changes, a new fingerprint is minted and the old version tiers down (T1 → T2 → T3). ADAM never deletes, only tiers down.

---

## Directory Structure

```
indeterminate-emergence/
├── CLAUDE.md                        # This file — execution contract
├── KNOWN_ISSUES.md                  # Active bugs and known limitations
├── CHANGELOG.md                     # Version history
├── LICENSE-CODE                     # MIT (for code)
├── LICENSE-PAPER                    # CC BY 4.0 (for paper and blog)
├── README.md                        # Public-facing overview
├── .gitignore
├── paper/
│   ├── indeterminate-emergence-v1.md    # Revised paper (source of truth)
│   ├── indeterminate-emergence-v1.tex   # LaTeX for arXiv submission
│   ├── indeterminate-emergence-v1.pdf   # Rendered PDF
│   └── figures/                         # Any diagrams or charts
├── blog/
│   └── what-if-security-meant-non-existence.md  # Blog post
├── poc/
│   ├── README.md                    # PoC overview, setup, usage
│   ├── requirements.txt             # Python dependencies (pinned)
│   ├── src/
│   │   ├── __init__.py
│   │   ├── proxy.py                 # FastAPI intent interface
│   │   ├── executor.py              # Sealed executor (normal + absorption)
│   │   ├── channel_shaper.py        # Response padding, timing, sizing
│   │   ├── accountant.py            # Privacy budget tracker
│   │   └── config.py                # Capability tokens, session config
│   ├── eval/
│   │   ├── __init__.py
│   │   ├── divergence_test.py       # Experiment 1: distribution comparison

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cerberusxops360) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
