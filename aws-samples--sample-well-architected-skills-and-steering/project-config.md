---
trigger: always_on
description: Steering file for agents and peers working in this repository. Covers contribution workflows, release process, open source community standards, and data hygiene rules.
---


# Agent Steering — WA Skills & Steering Docs

## Deprecation status

This repository's skill is deprecated in favor of the actively maintained
`aws-well-architected-review` skill in
[Agent Toolkit for AWS](https://github.com/aws/agent-toolkit-for-aws)
([product page](https://aws.amazon.com/products/developer-tools/agent-toolkit-for-aws/)).
See [SUCCESSOR_HANDOFF.md](SUCCESSOR_HANDOFF.md) and
[issue #147](https://github.com/aws-samples/sample-well-architected-skills-and-steering/issues/147)
for the migration plan. Practical implications for anyone working in this repo:

- **New skills and features are not being accepted.** Do not open PRs adding
  a new skill under `skills/` or new steering guidance.
- **Bug fixes, security reports, and corrections are still welcome** and follow
  the normal PR process below.
- The "release after every merged PR" rule and the SLA table further down
  still apply to accepted fix PRs, but are no longer a commitment for new
  feature work, since none is being merged.
- The evaluation harness under `evals/` keeps running as the
  compatibility/regression suite `SUCCESSOR_HANDOFF.md` calls for.

## Overview

Guidance for any agent or peer working in this repository. Read this before adding a skill, editing a steering file, merging a PR, or responding to a community issue.

## Usage

Read this file before:

- Adding or modifying a skill under `skills/`
- Editing a steering file under `steering/`
- Merging a pull request
- Responding to an issue or pull request from an external contributor
- Running evaluations

## Instructions

### Tooling

Always use `uv` to run Python scripts. Never use bare `pip`, `python -m venv`, or `python` directly.

```bash
uv run python run.py --skill my-new-skill --verbose
```

### Adding a New Skill

Skills must conform to the [Agent Skills Open Specification](https://agentskills.io/specification).

1. Create a directory under `skills/` with a descriptive kebab-case name (e.g., `skills/my-new-skill/`).
2. Add a `SKILL.md` file following the format in `skills/example-skill/SKILL.md`:
   - YAML frontmatter: `name`, `description`, `version`, `tags`
   - Sections: Overview, Usage, Instructions, Troubleshooting, Supporting Files, Core Concepts, Quick Reference, Common Mistakes
   - Cross-references to relevant sections in `steering/well-architected.md`
3. Add evaluations in `skills/my-new-skill/evals/evals.json`:
   - At least 3 test cases with realistic user prompts
   - 5–7 concrete PASS/FAIL assertions per case
   - Cover critical gaps, WA baselines, and edge cases
   - Run evals before opening a PR: `uv run python run.py --skill my-new-skill --verbose`
4. Open a PR describing what the skill does and which WA pillar(s) it covers.

### Modifying a Steering File

1. Edit files under `steering/`.
2. Guidance must be actionable and specific to the AWS Well-Architected Framework.
3. Open a PR explaining what changed and why.

### Pull Request Process

1. Fork the repository and create a feature branch.
2. Verify changes render correctly as Markdown.
3. Submit a PR with a short title, a description of what changed, and which WA pillar(s) are covered.
4. At least one project team member must review every PR. Large features should have additional reviewers.
5. After a PR merges, create a new GitHub release (see below).

### GitHub Releases

Create a new GitHub release after every merged PR — no exceptions, including small changes.

- Tag format: `vMAJOR.MINOR.PATCH`
  - `PATCH` — documentation fixes, typo corrections, minor clarifications
  - `MINOR` — new skills, new steering guidance, new evals
  - `MAJOR` — breaking changes to skill format or steering structure
- Release notes must list what was added, changed, or removed.

### Design Principles

- **Review and guidance, not code mutation.** Skills produce findings, plans, controls, or visual artifacts — never a PR-ready diff applied to the user's codebase. Keep the user in control of implementation decisions.
- **Aligned, not compliant.** Do not use "compliant" or "compliance" as customer outcomes. Use "aligned with best practices," "adherent to WA guidance," or similar phrasing.
- **Occam's razor.** Prefer the simplest skill design that satisfies the requirement. Avoid adding steps that do not add measurable value.
- **Data-driven.** Findings must be grounded in evidence from the user's codebase or configuration. Cite the specific resource, file, or configuration that supports each finding.

### Style Guidelines

- Use clear, imperative language (e.g., "Evaluate whether…" not "You might want to evaluate…").
- Include severity labels for findings: 🔴 High Risk, 🟡 Medium Risk, 🟢 Improvement.
- Reference specific AWS services where applicable.
- Keep steps concise — each step should represent a distinct action or evaluation.

### Open Source Community Standards

Respond to all issues and pull requests on schedule:

| Situation | Target |
|---|---|
| New issue or PR opened | Review within 1 business day |
| Security vulnerability | Respond immediately |
| General issues / PRs | Respond within 1 week |
| Non-urgent resolution | Resolve within 3 weeks |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-well-architected-skills-and-steering](https://github.com/aws-samples/sample-well-architected-skills-and-steering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
