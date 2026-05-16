---
trigger: always_on
description: Han is a Claude Code plugin: a suite of skills and agents for solo (or small-team) product engineers. It packages evidence-based planning, deep code review, investigation, and documentation workflows into deterministic slash commands that dispatch specialist sub-agents to do the judgment-heavy work.
---

# han: Project Map

Han is a Claude Code plugin: a suite of skills and agents for solo (or small-team) product engineers. It packages evidence-based planning, deep code review, investigation, and documentation workflows into deterministic slash commands that dispatch specialist sub-agents to do the judgment-heavy work.

Current version: **2.2.0** (see [CHANGELOG.md](./CHANGELOG.md)).

## Repository layout

```
/                       # repo root
├── README.md           # End-user landing page
├── CONTRIBUTING.md     # Contributor guide
├── CLAUDE.md           # This file
├── CHANGELOG.md    # Version history
├── .claude-plugin/
│   └── marketplace.json   # Test Double marketplace manifest
├── plugin/             # The actual plugin shipped to Claude Code
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── agents/         # 21 agent definitions (.md with frontmatter)
│   ├── skills/         # 15 skill directories, each with SKILL.md + references/
│   └── references/     # Cross-skill reference files (e.g. yagni-rule.md)
├── docs/               # Operator-facing documentation
│   ├── writing-voice.md   # Voice profile every doc follows
│   ├── concepts.md
│   ├── quickstart.md
│   ├── sizing.md
│   ├── yagni.md
│   ├── agents/         # Long-form docs for all 21 agents, plus README
│   ├── skills/         # Long-form docs for all 15 skills, plus README
│   ├── guidance/       # Contributor-facing authoring guidance
│   └── templates/      # Templates and coverage rule for long-form docs
└── images/             # Banner and graphics for README
```

The plugin is shipped from `plugin/`; documentation lives in `docs/`. Long-form docs in `docs/skills/{name}.md` and `docs/agents/{name}.md` are the canonical operator-facing source for every skill and every agent. The underlying definition (`plugin/skills/{name}/SKILL.md` or `plugin/agents/{name}.md`) is the implementation.

## When to use which doc

### Entry points

- **[README.md](./README.md).** End-user landing page. Use to understand what the plugin is and where to start. Lists install instructions and pointers to every other doc.
- **[CONTRIBUTING.md](./CONTRIBUTING.md).** Contributor guide for adding or editing skills, agents, and documentation. Read before changing any file under `plugin/` or `docs/`.
- **[CHANGELOG.md](./CHANGELOG.md).** Version history. Check when a behavior or skill name in user-supplied context doesn't match what's on disk. May be a pre-2.0 rename or a removed feature.

### Writing voice

- **[docs/writing-voice.md](./docs/writing-voice.md).** Voice profile every doc in the plugin follows. No em-dashes, direct second person, plainspoken mentor tone, named voice violations to avoid.

### Core mental model (`docs/`)

- **[docs/concepts.md](./docs/concepts.md).** The skill-vs-agent model that runs through the whole plugin. Read once before doing anything else. Every other doc assumes this vocabulary.
- **[docs/quickstart.md](./docs/quickstart.md).** Four path-based recipes (plan a feature, investigate a bug, review code, set up a project). Use when picking which skill to run for a specific situation.
- **[docs/sizing.md](./docs/sizing.md).** The small / medium / large dispatch model used by the five swarming skills (`/code-review`, `/gap-analysis`, `/iterative-plan-review`, `/plan-a-feature`, `/plan-implementation`). Use when a swarming skill needs to decide team size, or when a user asks what `medium` / `large` mean.
- **[docs/yagni.md](./docs/yagni.md).** The evidence-based "You Aren't Gonna Need It" rule every planning, review, and architecture skill applies before committing items to its artifact. Use when explaining why an item was deferred or rejected from a plan / review / ADR.

### Skill catalog (`docs/skills/`)

- **[docs/skills/README.md](./docs/skills/README.md).** Index of all 15 skills grouped by purpose (planning, investigation, review, discovery, conventions, reporting). Start here when looking for the right slash command.
- **[docs/skills/plan-a-feature.md](./docs/skills/plan-a-feature.md).** Spec a feature from scratch through an evidence-based interview that walks the design tree and dispatches specialist reviewers.
- **[docs/skills/plan-implementation.md](./docs/skills/plan-implementation.md).** Turn a feature specification into an implementation plan through a project-manager-led team conversation.
- **[docs/skills/plan-a-phased-build.md](./docs/skills/plan-a-phased-build.md).** Split a body of context (gap analysis, PRD, design doc) into a numbered sequence of vertical-slice phases, each independently demoable.
- **[docs/skills/iterative-plan-review.md](./docs/skills/iterative-plan-review.md).** Stress-test an existing plan through multiple codebase-grounded review passes. Edits the plan in place and records every finding.
- **[docs/skills/investigate.md](./docs/skills/investigate.md).** Evidence-based investigation of bugs, failures, and unexpected behavior, with adversarial validation of the proposed fix.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [testdouble/han](https://github.com/testdouble/han) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
