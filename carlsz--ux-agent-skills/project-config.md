---
trigger: always_on
description: Instructions for AI agents working **on this repo** — how it's laid out, how to author new
---

# AGENTS.md

Instructions for AI agents working **on this repo** — how it's laid out, how to author new
components, and which rules the tests actually enforce.

> Working on a *host app* with this plugin installed? You want [README.md](README.md). This file is
> for changing the plugin itself.

This is a Claude Code plugin. It ships no application code — every component is Markdown, and the
Python under [`scripts/`](scripts/) and [`tests/`](tests/) exists to keep that Markdown honest.

## Composition — three layers

This plugin inherits the composition rule from [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills):

- **Personas** (`agents/*.md`) — roles with a perspective and an output format. *The who.*
- **Skills** (`skills/<name>/SKILL.md`) — workflows with steps and exit criteria. *The how.*
- **Slash commands** (`commands/*.md`) — user-facing entry points. *The when.*

Keep the layers honest: a persona holds the point of view and never inlines a workflow; a skill holds
the steps and its own `references/`; a command stays thin — parse args, invoke persona + skill,
nothing more. When you're unsure where something belongs, ask which of *who / how / when* it answers.

## Repo layout

```
ux-agent-skills/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest — metadata only, no component lists
│   └── marketplace.json    # marketplace catalog (self-lists this plugin)
├── agents/                 # personas — the who (usability-auditor, cuj-author, cuj-auditor)
├── skills/                 # skills — the how
│   ├── usability-audit/        # usability audit workflow + framework lenses & report contract
│   ├── ux-audit/               # the suite roll-up (fan-out + normalize + verdict)
│   ├── spec-cuj/               # author critical user journeys by interview (+ cuj-contract)
│   ├── audit-cuj/              # replay journeys, report the step that broke
│   └── render-report/          # render reports to self-contained HTML (the one non-auditor skill)
├── commands/               # slash commands — the when
├── scripts/                # validate_report.py, validate_cuj.py, audit_safety.py, render_report_html.py
├── tests/                  # contract / component / cuj-contract / safety / docs / evals / render-html checks
├── evals/                  # trigger + behavioral evals (Sprout as the target)
└── SPEC.md                 # this repo's design spec (§1–8 usability auditor, §9 CUJs, §10 walk-through + HTML) — what/why
```

**Components are auto-discovered by convention** from `agents/`, `skills/*/SKILL.md`, and
`commands/`. Adding one needs **no `plugin.json` edit** — the manifest carries metadata and
`dependencies` only. Touch it to bump `version`, or to add an external plugin you wrap (mirror any
`dependencies` change into `marketplace.json`).

## Frontmatter by layer

Every component is YAML frontmatter + a Markdown body. The fields are few and the `description` does
real work — it's the **routing corpus** (see [Triggers and routing](#triggers-and-routing)), so write
it for a matcher, not just a reader.

| Layer | Path | Frontmatter |
|-------|------|-------------|
| Persona | `agents/<name>.md` | `name`, `description` |
| Skill | `skills/<name>/SKILL.md` | `name`, `description` |
| Command | `commands/<name>.md` | `name`, `description`, `argument-hint` |

There is no separate `triggers:` key — **trigger phrases live inside `description`**, in prose. Copy
the shape from [`agents/usability-auditor.md`](agents/usability-auditor.md):

```yaml
description: Senior UX auditor that runs expert heuristic evaluation of a host app and writes a
  severity-scored report to .ux/audits. Trigger phrases include "usability audit", "heuristic
  evaluation", "UX audit", "usability review".
```

`name` must match the file (persona/command) or the **directory** (skill) — `test_evals.py` keys off
the skill's directory name, so a mismatch fails CI.

## Adding a new auditor, end to end

The suite is deliberately uniform: every auditor emits the same
[report contract](skills/usability-audit/references/report-contract.md), so a new one is mostly
conformance work. In order:

1. **`agents/<x>-auditor.md`** — the persona. Body must state the findings-only boundary, the
   render-vs-source honesty rule, the 0–4 severity format, and point at its frameworks.
2. **`skills/<x>-audit/SKILL.md`** — the workflow. Numbered steps with **explicit exit criteria**.
   Must cover static/live/hybrid modes, writing under `.ux/audits/`, and the coverage-gap rule.
3. **`skills/<x>-audit/references/`** — the framework lenses, self-contained. Personas point here;
   they don't inline heuristics.
4. **`commands/<x>-audit.md`** — thin entry point. Document `target`, `scope`, `mode`.
5. **`evals/cases/<x>-audit.json`** — **mandatory, see the trap below.** `skill_name` matching the
   directory, ≥3 positive triggers, ≥2 negatives (each naming an `owner` that exists in
   `evals/cases/competitors.json`), ≥1 behavioral eval with `id` / `prompt` / `expected_output` /
   `expectations`.
6. **`tests/test_components.py`** — hand-write a `check_<x>()` and wire it into `main()`.
   **See the trap below.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlsz/ux-agent-skills](https://github.com/carlsz/ux-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
