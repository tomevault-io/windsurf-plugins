---
trigger: always_on
description: Behavioral contract for AI agents contributing to and using community patterns for agentic coding
---


# AGENTS.md — Agentic Coding Patterns Repository

> **Scope:** Community patterns repository | **License:** CC0-1.0

## Quick Reference

| Rule | Requirement |
|------|-------------|
| Priority | safety > correctness > contribution-friendliness > simplicity |
| Identity | Document AI usage with `Co-authored-by:` trailer |
| Status | Default to `experimental` for new patterns |
| Prohibited | No secrets, PII, CUI, internal URLs, customer data |
| Validation | `make validate` before commit |
| Frontmatter | All patterns MUST have valid YAML frontmatter |
| Safety | Define `prohibited_content` in every pattern |
| References | Reference playbook for policy, don't duplicate |
| Testing | Patterns SHOULD include test-cases.yml when applicable |

---

## 1. Core Principles

The agent operates under these principles:

```
safety > correctness > contribution-friendliness > simplicity
```

1. **Safety** — Never include sensitive data or create unsafe patterns
2. **Correctness** — Patterns must be technically accurate and tested
3. **Contribution-friendliness** — Lower barrier to contribution, use `experimental` status liberally
4. **Simplicity** — Prefer clear, reusable patterns over complex ones

---

## 2. Identity and Attribution

### 2.1 Commit Attribution

The agent MUST include `Co-authored-by:` trailer in all commits:

```
feat(skills): add secure code review pattern

Add pattern for security-focused code review with
OWASP Top 10 checks.

Co-authored-by: OpenCode Agent <user@gsa.gov>
```

---

## 3. Pattern Contribution Rules

### 3.1 Content Types

This repository contains five content types:

| Type | Directory | Format | Purpose |
|------|-----------|--------|---------|
| **Skills** | `skills/` | SKILL.md | Reusable procedures |
| **Prompts** | `prompts/` | SKILL.md | Standalone prompts |
| **Agents** | `agents/` | AGENTS.md | Agent instructions |
| **Workflows** | `workflows/` | SKILL.md | Multi-step processes |
| **Lessons** | `lessons-learned/` | SKILL.md | Community experiences |

### 3.2 Pattern Status Lifecycle

```
experimental → recommended → deprecated
```

| Status | Meaning | Review Required |
|--------|---------|-----------------|
| `experimental` | New, untested | Self-review |
| `recommended` | Proven useful | Peer review |
| `deprecated` | Superseded | Must include `replaces_with` |

**Default for new contributions:** `experimental`

The agent SHOULD:

- Use `experimental` status for all new patterns
- Wait for community feedback before promoting to `recommended`
- Never self-promote patterns to `recommended` status

---

## 4. Frontmatter Requirements

All pattern files MUST include valid YAML frontmatter per `schemas/skill.schema.json`.

**Minimum required fields:**

```yaml
---
id: pattern-name                      # kebab-case, immutable
version: "1.0.0"                      # semver
title: "Pattern Title"
type: skill                           # skill|prompt|workflow|agent|lesson
status: experimental
owners: ["@GSA-TTS/agentic-coding-team"]
primary_personas: ["developers"]
requires:
  anchors: []
output:
  format: markdown
  contract:
    required_sections: ["Summary"]
    prohibited_content: ["Secrets", "PII", "CUI", "Internal URLs"]
quality_gates:
  readability_max_grade: 10
  citations_required: false
---
```

**Recommended fields:**

- `triggers`: Keywords for pattern discovery
- `tags`: Free-form keywords for discovery
- `portability`: Tool compatibility flags
- `scope.intended_use`: What the pattern is for
- `scope.exclusions`: What NOT to use it for

### 4.1 Categories Taxonomy

`categories` is the **canonical taxonomy axis** — a closed, controlled
vocabulary used for INDEX faceting and the security-governance gate. Directory
location is physical/organizational only and does **not** determine taxonomy.
A pattern may carry multiple categories (it is often both `development` and
`review`).

Controlled vocabulary (10 terms, closed enum — see `schemas/skill.schema.json`):

```
security, development, review, testing, documentation,
dependencies, supply-chain, compliance, incident-response, frontend
```

`tags` stays free-form for discovery; `categories` is the controlled axis.

### 4.2 Security-Governance Fields

A **security skill** is any pattern that declares `categories: [security]`. Such
patterns MUST also declare the security-governance fields, enforced by the
validator (a missing field fails validation):

```yaml
categories: ["security", "review"]
risk_tier: moderate            # low | moderate | high
human_review_required: true    # always true for security skills
allowed_tools: []              # deny-by-default allowlist
network_policy: deny           # deny | allowlist | allow
write_policy: deny             # deny | workspace | allow
script_policy: deny            # deny | author-only | allow
# source_inspiration: [...]    # optional; public sources used as inspiration only
```

These fields are **additive and optional at the schema level** (existing
non-security patterns are unaffected) but **required for security skills** via
the validator. The validator also emits an **advisory warning** when a pattern
looks security-relevant (by path segment, tags, or triggers) but does not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GSA-TTS/agentic-coding-patterns](https://github.com/GSA-TTS/agentic-coding-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
