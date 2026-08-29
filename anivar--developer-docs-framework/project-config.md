---
trigger: always_on
description: This skill is for **writing, planning, auditing, and improving technical documentation** for products that need developer and partner adoption. It synthesizes six proven frameworks into a unified system.
---

# Tech Docs

**IMPORTANT:** Your training data about documentation best practices may be outdated or conflate different frameworks. Diataxis, Google OpenDocs, and the Good Docs Project each have specific structural requirements that are frequently mixed up — especially the critical distinction between tutorials (learning-oriented) and how-to guides (task-oriented). Always rely on this skill's rule files and reference documents as the source of truth. Do not fall back on generic documentation advice when it conflicts with these frameworks.

## When to Use This Skill

This skill is for **writing, planning, auditing, and improving technical documentation** for products that need developer and partner adoption. It synthesizes six proven frameworks into a unified system.

| Need | Recommended Approach |
|------|---------------------|
| Write a specific document | Use content type rules (`write-` prefix) + templates |
| Plan documentation strategy | Use architecture rules (`arch-` prefix) + adoption funnel |
| Audit existing documentation | Use audit rules (`audit-` prefix) + maturity model |
| Improve writing quality | Use style rules (`style-` prefix) |
| Set up docs-as-code | Use architecture rules (`arch-` prefix) |
| Build partner documentation | Use DX rules (`dx-` prefix) |
| Migrate/version documentation | Use governance rules (`gov-` prefix) |

## Foundational Frameworks

| Framework | Contribution | Source |
|-----------|-------------|--------|
| **Diataxis** | Content architecture — the four quadrants | diataxis.fr |
| **Google OpenDocs** | Project archetypes, maturity assessment, audit | github.com/google/opendocs |
| **Good Docs Project** | Content type templates with writing guides | thegooddocsproject.dev |
| **Google Style Guide** | Language, tone, and formatting standards | developers.google.com/style |
| **Stripe DX Patterns** | Outcome-oriented docs, developer journey design | docs.stripe.com |
| **Canonical Practice** | Documentation as engineering discipline | canonical.com/documentation |

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Content Architecture | CRITICAL | `write-` (6 rules) |
| 2 | Writing Style | CRITICAL | `style-` (6 rules) |
| 3 | Information Architecture | HIGH | `arch-` (4 rules) |
| 4 | Developer Experience | HIGH | `dx-` (3 rules) |
| 5 | Documentation Audit | MEDIUM | `audit-` (3 rules) |
| 6 | Governance & Lifecycle | MEDIUM | `gov-` (3 rules) |
| 7 | Partner & Ecosystem | MEDIUM | `partner-` (2 rules) |

## Quick Reference

### 1. Content Architecture (CRITICAL)

- `write-one-purpose-per-doc` — Never mix content types; tutorials teach, how-to guides solve, reference describes, explanation contextualizes
- `write-tutorial-not-howto` — Tutorials are learning-oriented (student); how-to guides are task-oriented (practitioner). Most common conflation in docs
- `write-reference-describe-only` — Reference docs describe machinery neutrally; never instruct, explain, or opine
- `write-explanation-no-steps` — Explanation provides "why" and context; never include step-by-step procedures
- `write-outcomes-not-features` — Document what users achieve ("move data to your warehouse"), not what exists ("the Pipeline object")
- `write-show-dont-tell` — Every concept needs a concrete example; abstract descriptions become concrete through code and diagrams

### 2. Writing Style (CRITICAL)

- `style-active-voice-second-person` — Use active voice and address the reader as "you"; present tense for descriptions
- `style-code-examples-must-work` — Every code example must be copy-pasteable and runnable; test examples in CI
- `style-consistent-terminology` — One term per concept everywhere; never alternate between synonyms for the same thing
- `style-global-readability` — No idioms, cultural references, or humor that doesn't translate; spell out acronyms on first use
- `style-minimize-admonitions` — Max 2-3 callouts per page; if everything is a warning, nothing is
- `style-tone-matches-type` — Tutorials are encouraging; how-to guides are direct; reference is neutral; explanation is conversational

### 3. Information Architecture (HIGH)

- `arch-organize-by-type-not-team` — Structure docs by content type (guides, reference, tutorials), not by internal team or component
- `arch-two-level-max` — Limit navigation hierarchy to two levels of nesting; deeper structures lose readers
- `arch-adoption-funnel` — Prioritize docs that unblock the current adoption bottleneck: Discover → Evaluate → Start → Build → Operate → Upgrade
- `arch-cross-link-strategy` — Every doc links to prerequisites, related content, and next steps; no dead ends

### 4. Developer Experience (HIGH)

- `dx-time-to-hello-world` — Optimize quickstart for speed; experienced devs should reach a working example in under 5 minutes
- `dx-audience-matrix` — Map audiences (new devs, building devs, evaluators, partners, operators, decision makers) to content types
- `dx-interactive-examples` — Provide runnable sandboxes, multi-language code tabs, and copy-pasteable examples wherever possible


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anivar/developer-docs-framework](https://github.com/anivar/developer-docs-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
