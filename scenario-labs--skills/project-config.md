---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repository.
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repository.

## What this repo is

Public Agent Skills for [Scenario](https://scenario.com): procedural knowledge that teaches AI coding agents how to create production-ready content (images, video, audio, textures, skyboxes, 3D, custom models) through the [Scenario MCP server](https://mcp.scenario.com). The workflows serve games, entertainment, and any creative vertical. Skills follow the [Agent Skills](https://agentskills.io) format, install with `npx skills add scenario-labs/skills`, and are listed on [skills.sh](https://www.skills.sh/scenario-labs/skills).

## Layout

```
skills/<name>/SKILL.md   # name must equal the directory name
tests/<name>/            # test suite for any script shipped with skill <name>
```

Supporting files (heavy references, scripts) may sit next to a SKILL.md only when the content is too large to inline. Link supporting files directly from SKILL.md: agents resolve file references one level deep, so a reference chained through another supporting file may never be read.

A skill may also carry a `README.md` documenting how it was built and why it makes the choices it does. That file is for the agents and humans working on the skill, not for the agent running it, so it is the one supporting file exempt from the link rule (`pnpm skill-files` skips it): linking it would spend body words and invite a runtime agent to read maintainer notes as instructions, which has been observed to make an agent discount a fact it needed.

Every script shipped with a skill has a test suite in `tests/<name>/` at the repo root, never inside the skill directory: published skill directories carry only what an agent needs at runtime. Python scripts are tested with stdlib `unittest`; TypeScript scripts with vitest. See Validation and testing.

## Public content only

This repository is public. Everything in it, including commit messages, PR text, and issue text, must be limited to publicly shareable language:

- Reference only public surfaces: scenario.com, app.scenario.com, help.scenario.com (the Scenario Knowledge Base), mcp.scenario.com and its `/docs`, docs.scenario.com, and the public model catalog.
- Never reference internal repositories, source file paths, internal hostnames or environments, internal project or team names, customer names, real team/project/API identifiers, credentials, pricing internals, or unreleased features.
- Facts must be verifiable from public surfaces (the tool reference at mcp.scenario.com/docs/tools, live public catalog searches). If a fact is only knowable from internal sources, leave it out.
- When in doubt, treat it as internal: ask, or drop it. Example: no internal repository naming.

## Authoring contract

CI enforces the mechanical parts of this contract on every push and PR: [`skills-ref validate`](https://github.com/agentskills/agentskills/tree/main/skills-ref) (the Agent Skills reference validator) for the spec rules, plus house-style greps, formatting (prettier), spell checking (cspell), and supporting-file checks. Run the content checks locally with `pnpm run validate` (commit messages and PR titles are linted separately with commitlint), or spec-validate a single skill with:

```bash
uvx --from "git+https://github.com/agentskills/agentskills.git#subdirectory=skills-ref" skills-ref validate skills/<name>
```

- Frontmatter: `name`, `description`, and `license: MIT`, nothing else. The spec caps `name` at 64 characters and `description` at 1024; the other spec-optional fields (`compatibility`, `metadata`, `allowed-tools`) are not used in this repo.
- `name`: lowercase letters, numbers, and hyphens; must equal the directory name.
- `description`: third person, starts with "Use when", describes triggering conditions only (never a summary of the skill's workflow), under 500 characters, rich in keywords an agent would search for.
- Body: 600-900 words is the house target; the build fails past 1400 words or 500 body lines (`pnpm style`). Structure: Overview, Quick reference, one excellent worked example, Common mistakes.
- Measure before trimming: `awk '/^---$/{c++; next} c>=2' skills/<name>/SKILL.md | wc -w` is the exact number `pnpm style` checks, and `wc -l` the same for the 500-line cap.
- Why the budget: agents load only `name` and `description` at startup; the body enters context only when the skill triggers, and then every word competes with the user's task. Spend words on facts an agent would otherwise guess wrong, not on prose.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scenario-labs/skills](https://github.com/scenario-labs/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
