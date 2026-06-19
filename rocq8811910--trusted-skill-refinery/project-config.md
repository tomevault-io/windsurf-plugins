---
trigger: always_on
description: Trusted Skill Refinery
---

# Trusted Skill Refinery

---

## English

### Name

Trusted Skill Refinery

### Purpose

This project provides a safety-first pipeline for **fetching, auditing, sanitizing, validating, and publishing** clean AI agent skills from untrusted external sources. It processes third-party skills through controlled fetch, isolation, static audit, human review, sanitization planning, non-executable candidate generation, validation, and admission.

**v0.2-dev** adds a controlled auto-rewrite engine for low-risk prompt/document skills and a high-risk routing system for scraper/API/browser skills.

### Core Principle: Dark Skill Assumption

The project is based on the **Dark Skill Assumption**: every external AI agent skill should be treated as potentially unsafe until it has passed controlled fetching, isolation, audit, risk scoring, human review, validation, and admission.

This does not mean every skill is malicious. It means no external skill should be trusted by default.

Downloading a skill does not mean trusting it. Auditing does not mean admitting it to your library. Sanitization does not mean it is safe to execute. Admission requires validation and human approval.

### Version Scope

#### v1.0 (Current Release)

v1.0 focuses on the manual sanitization workflow. It supports:

- Controlled fetching from GitHub archive URLs
- Safe archive inspection with path traversal protection
- Raw skill auditing (static, dependency, instruction)
- Risk decision (R1-R4 classification)
- Rewrite/sanitization planning
- Non-executable candidate generation
- Static validation
- Sanitized library admission
- Optional non-executable AgentFactory export packages

v1.0 does **not** provide fully automatic executable rewriting. Rewrite support in v1.0 is conservative, planning-oriented, and non-executable by default.

#### v0.2-dev (Development Branch)

v0.2-dev introduces two major new systems on top of v1.0:

**Controlled Auto-Rewrite Engine** (for low-risk prompt/document skills):
- Rule-based detection and removal of dangerous instruction patterns
- Rewrite diff — auto-generated comparison between raw and rewritten skill
- Functional preservation report — quantifies safe content retained vs removed
- Removed risk report — documents what was removed and why
- Safety regression gate — automated check that rewrite did not introduce new risks
- Streamlined human review for auto-rewritten candidates
- Non-executable sanitized library admission

**High-Risk Routing System** (for scraper/API/browser skills):
- High-risk skill type detection and classification
- Static audit (19 checks per skill covering API keys, network, browser, credentials)
- Risk routing: quarantine or manual wrapper planning
- 6 cross-cutting policies: sandbox, API key handling, rate limiting, domain allowlisting, data staging, approval gates (10 gates)
- Future wrapper roadmap (stages H1-H5 from design through controlled test)

v0.2-dev does **not** support:
- Automatic rewriting of scraper/API/browser/credential skills
- Executable wrapper generation
- Execution of raw or rewritten skills
- Dependency installation
- API key reading
- Automatic AgentFactory export

### Low-Risk Route (prompt/document skills)

```
Prompt/Document Skill
  → Controlled Intake
  → Static Audit
  → Auto-Rewrite Candidate
  → Rewrite Diff
  → Functional Preservation Report
  → Removed Risk Report
  → Safety Regression Gate
  → Human Review
  → Admission Package
  → Final Human Approval
  → Non-Executable Sanitized Library Entry
```

This route is for skills that are purely instructional — no scripts, no dependencies, no network requirements, no API keys. The auto-rewrite engine removes dangerous instruction patterns (prompt injection, hidden instructions, secret extraction, local file scan, arbitrary network access) while preserving safe content.

**Current v0.2-dev result**: 2 synthetic examples admitted (prompt_skill_001_clean_v0_2, document_skill_001_clean_v0_2). Both non-executable, all permissions false.

### High-Risk Route (scraping/API/browser/crawler skills)

```
Scraping/API/Browser/Crawler Skill
  → Controlled Intake
  → Static Audit (19 checks)
  → Risk Routing
  → Quarantine (R4: browser automation, credential management)
  → OR Manual Wrapper Planning (R3: API scrapers)
  → Sandbox / API Key / Domain Allowlist / Rate Limit / Data Staging Policy
  → No Executable Output by Default
```

This route applies when a skill requires network access, API keys, browser automation, or credential management. These skills are **never** auto-rewritten. They are either quarantined permanently (browser, credential) or deferred for manual wrapper planning (API scrapers).

**Current v0.2-dev result**: 5 real Firecrawl skills stress-tested. 0/5 allowed auto-rewrite. 2 quarantined (browser automation, credential management). 3 deferred for manual wrapper planning.

### What Must Never Be Automatic

The following must **never** be automated, in any version:

- Execution of raw or rewritten skills
- Dependency installation
- API key reading or storage
- Network access during audit or rewrite
- Browser session access
- Credential file (.env) writes
- AgentFactory export without final human approval
- Sanitized library admission without validation + human approval
- Trust in auto-rewrite output without human review


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rocq8811910/trusted-skill-refinery](https://github.com/rocq8811910/trusted-skill-refinery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
