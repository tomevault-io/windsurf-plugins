---
trigger: always_on
description: |
---


# Project Bootstrapper

> **Philosophy**: Define how code must be written before writing any code. Bugs are prevented at design time, not discovered at runtime.

This is a **meta-skill** — it does not write application code. It generates the **rules, patterns, guardrails, and quality standards** that govern all code written afterward, by any developer or AI assistant.

## How It Works

```
[Idea] → [Interview] → [Tech Stack] → [Skill Map] → [Generate Skills] → [Validate] → [Code]
```

## Activation

This skill activates when:
- User describes a new software project idea
- User says "bootstrap", "new project", "start from scratch", "set up project"
- User wants to generate a skill suite for an existing codebase
- User asks for coding standards, project scaffolding, or development guardrails
- Any context where skills should be created before development begins

## Important: Read References First

Before generating ANY skills, you MUST read these reference files in order:
1. `references/skill-catalog.md` — Full catalog of 40+ skill domains
2. `references/skill-template.md` — Universal template every skill must follow
3. `references/generation-guide.md` — Domain-specific generation instructions with code
4. `references/quality-standards.md` — Quality checklist for generated skills
5. `references/cross-cutting-concerns.md` — Rules that span all skills

---

## Phase 1: Project Intelligence Gathering

### 1.1 — Understand the Idea

Extract or ask about:

**What** (Product):
- What does this project do? (one sentence)
- What type is it? (web app, mobile app, desktop app, CLI, library/SDK, API service, browser extension, IoT, embedded, game, data pipeline, ML platform, monorepo with multiple products)
- Who is the end user? (developers, consumers, enterprise, internal team)
- What is the revenue model? (open-source, freemium, SaaS subscription, one-time purchase, marketplace, ad-supported, enterprise license)

**How Big** (Scale):
- Expected users at launch? At 12 months?
- Data volume? (records, files, events/sec, storage)
- Geographic scope? (single region, continental, global)
- Availability requirement? (99.9%, 99.95%, 99.99%)

**How** (Constraints):
- Required technologies? (must use React, must deploy on AWS, etc.)
- Existing codebase? (greenfield vs brownfield)
- Team size? (solo, small team 2-5, medium 5-15, large 15+)
- Timeline? (hackathon/weekend, MVP in weeks, production in months)
- Budget constraints? (free tier only, moderate, enterprise)
- Compliance requirements? (GDPR, HIPAA, SOC2, PCI-DSS, COPPA, CCPA)

**If the user already provided details**, extract answers from their message instead of asking. Only ask what's missing and genuinely needed to make tech stack decisions.

### 1.2 — Determine Tech Stack

Based on the answers, recommend a complete tech stack.

#### 🔍 MANDATORY: Version Research (Latest Stable) — ZERO TOLERANCE

**This is NON-NEGOTIABLE. Before proposing ANY technology, you MUST verify its latest stable version via real-time lookup.**

⚠️ **CRITICAL**: AI models have knowledge cutoffs. Package ecosystems evolve daily. A skill generated with outdated versions will produce vulnerable, deprecated code.

**Research Protocol** (execute for EVERY technology):
1. **Use available tools** (in priority order):
   - `WebSearch`: `"{package} latest stable version {current_year}"`
   - `WebFetch`: Official docs site (e.g., `nextjs.org`, `python.org`, `go.dev`)
   - Context7: `resolve-library-id` → `query-docs` for changelog
   - Package registry: npmjs.com, pypi.org, crates.io, pkg.go.dev, maven.apache.org

2. **Extract exact version**:
   - Format: `Major.Minor.Patch` (e.g., `Next.js 16.1.0`)
   - Verify it's STABLE (not alpha, beta, RC, canary, nightly)
   - Note release date — reject if >6 months old without updates

3. **Document verification**:
   ```
   Technology: Next.js
   Version: 16.1.0
   Verified via: nextjs.org/blog
   Verification date: 2026-03-09
   Release date: 2026-02-15
   Node requirement: >= 22.0.0
   ```

**HARDCORE RULES**:
- ✅ **MUST**: Verify EVERY dependency, not just frameworks
- ✅ **MUST**: Pin exact versions in all configs (`package.json`, `requirements.txt`, `Cargo.toml`, etc.)
- ✅ **MUST**: Use latest APIs/syntax from verified version in ALL code examples
- ❌ **NEVER**: Use memorized versions under ANY circumstance
- ❌ **NEVER**: Skip verification even for "well-known" packages
- ❌ **NEVER**: Use deprecated APIs from older versions
- ⚠️ **WARN**: If verification fails, mark clearly: `⚠️ VERSION UNVERIFIED — MUST CONFIRM`

**Abandonment Detection**:
- Last commit/release >12 months = investigate alternatives
- No maintainer response to issues >6 months = red flag
- Security advisories unpatched >30 days = DO NOT USE

**Example research queries**:
- `"next.js latest version"` → nextjs.org or npm
- `"postgresql latest stable release"` → postgresql.org
- `"tailwind css latest version"` → tailwindcss.com or npm
- Context7: resolve library ID → query docs for version/changelog

#### Tech Stack Decision Table

Organize as a layered decision table:

```
┌──────────────────────────────────────────────────────────────────────┐
│  TECH STACK PROPOSAL (versions verified: {date})                     │
├────────────────┬─────────────────────────────────┬───────────────────┤

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ersinkoc/project-bootstrap](https://github.com/ersinkoc/project-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
