---
trigger: always_on
description: Apply these keywords consistently in this document and the documents linked from this document.
---

# AGENTS.md

## Requirement Level Keywords

Apply these keywords consistently in this document and the documents linked from this document.

| Keyword | Synonym | Meaning |
| ------- | ------- | ------- |
| "MUST" | "REQUIRED" | Non-negotiable requirement; no exceptions. |
| "MUST NOT" |  | Non-negotiable prohibition; no exceptions. |
| "SHOULD" | "RECOMMENDED" | Strongly preferred; deviation is allowed only after weighing the implications. |
| "SHOULD NOT" | "NOT RECOMMENDED" | Strongly discouraged; allowed only after weighing the implications. |
| "MAY" | "OPTIONAL" | Genuinely optional; no preference implied. |

## Project Overview

- This is a web application project for generating human-friendly passwords.
- The target users are people who use non-QWERTY keyboard layouts.
- Generated passwords use only characters that are common to both the QWERTY keyboard layout and the user's selected keyboard layout.
- Passwords should remain practical to type on the user's keyboard while preserving clear generation rules.
- For tech stack, deployment target, npm run-scripts, and durable directory placement, consult [Project Structure](.agents/skills/project-structure/SKILL.md) and [Development Guidelines](.agents/skills/development-guidelines/SKILL.md).

## Skill Index

`AGENTS.md` is the master routing index for project skills. Consult the relevant skill before acting on matching work.

| Skill | When to apply |
| ----- | ------------- |
| [Agent Skills Best Practices](.agents/skills/agent-skills-best-practices/SKILL.md) | Creating, refining, splitting, renaming, deleting, or auditing project skills or this skill index |
| [Application Security Requirements](.agents/skills/application-security-requirements/SKILL.md) | Reviewing generated password exposure, Web Crypto, clipboard use, storage, URLs, telemetry, dependencies, environment variables, or privacy-sensitive behavior |
| [Code Review Guideline](.agents/skills/code-review-guideline/SKILL.md) | Reviewing a diff, pull request, local change, or post-implementation self-review |
| [Development Guidelines](.agents/skills/development-guidelines/SKILL.md) | Starting any task; implementing, refactoring, running commands, checking current docs, adding dependencies, changing verification behavior, writing commit messages, or opening pull requests |
| [E2E Testing Guidelines](.agents/skills/e2e-testing-guidelines/SKILL.md) | Writing, running, reviewing, or maintaining Playwright tests for generator workflows, clipboard feedback, layout selection, or responsive behavior |
| [Keyboard Layout Requirements](.agents/skills/keyboard-layout-requirements/SKILL.md) | Adding, editing, testing, or reviewing keyboard layout data, QWERTY intersections, character categories, or layout metadata |
| [Maintainable Code Guidelines](.agents/skills/maintainable-code-guidelines/SKILL.md) | Reviewing readability, naming, abstraction boundaries, complexity, dead code, or scope discipline |
| [Observability Guidelines](.agents/skills/observability-guidelines/SKILL.md) | Throwing, catching, reporting, or logging errors; console output; browser failure messages; telemetry boundaries; or debugging generated-password failures |
| [Password Generation Requirements](.agents/skills/password-generation-requirements/SKILL.md) | Implementing, testing, reviewing, or changing password generation, entropy, randomness, grouping, character pools, or output rules |
| [Performance and Reliability Requirements](.agents/skills/performance-and-reliability-requirements/SKILL.md) | Reviewing bundle weight, dependency restraint, browser API failure modes, local responsiveness, or runtime reliability |
| [Project Structure](.agents/skills/project-structure/SKILL.md) | Navigating the repository, locating files, placing modules, checking stack/deployment context, or updating durable directory conventions |
| [Quality Assurance Guidelines](.agents/skills/quality-assurance-guidelines/SKILL.md) | Reviewing verification evidence, test coverage, skipped checks, manual checks, lint/format proof, or residual risk |
| [React Component Guidelines](.agents/skills/react-component-guidelines/SKILL.md) | Writing, reviewing, or refactoring React components, Tailwind class usage, local state boundaries, accessible controls, or test locators |
| [Routing Guidelines](.agents/skills/routing-guidelines/SKILL.md) | Creating, moving, renaming, or reviewing TanStack Start routes, root metadata, router config, stylesheet links, or URL contracts |
| [UI Design Principles](.agents/skills/ui-design-principles/SKILL.md) | Designing, implementing, or reviewing user-facing surfaces, responsive behavior, accessibility, visual tone, copy, focus states, or feedback states |

## Response Approach

Use this workflow for single-agent work in this repository. The agent owns planning, implementation, investigation, verification, review, and reporting directly.

### Overall Strategy

Non-trivial work should move through the same decision sequence even when some steps are brief.

1. Classify the request and load the relevant project guidance.
2. Define success criteria, constraints, affected surface, dependencies, and verification expectations.
3. Inspect the smallest useful code and documentation context.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axross/human-keygen](https://github.com/axross/human-keygen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
