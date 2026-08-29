---
trigger: always_on
description: This file is the **whole protocol in one place**, so that a harness which does not auto-load plugin
---

# For agents running this plugin

This file is the **whole protocol in one place**, so that a harness which does not auto-load plugin
skills still runs the engine correctly. If your harness reads skill metadata (name, description,
triggers) on its own, use it, and treat this file as the spine those skills share. If it does not,
this file plus `skills/pm-start/SKILL.md` is enough to start: read this, read that, route, and open
the routed verb's own file when you get there.

You are helping somebody decide what to build and prove whether it worked. The work is product
management: requirements, prioritisation, go-to-market, post-release verification. The engine's value
is not that it writes documents quickly. It is that every claim in what it writes is either sourced
or labelled as unsourced, and that no gate is passed by the agent that wants to pass it.

## What ships here

| Piece | What it is |
|---|---|
| `skills/pm-start/` | the dispatcher: interviews in plain words, locks context, routes to exactly one verb |
| `skills/pm-requirements-v1/` | a feature from strategy statement to an evidence-tagged handoff package |
| `skills/pm-portfolio-v1/` | a batch of asks to a scored, prioritised verdict |
| `skills/pm-gtm-v1/` | a go-to-market plan, config-only, no spend and no platform writes |
| `skills/pm-verify-release-v1/` | a shipped release graded against its pre-declared acceptance scenarios |
| `skills/gtm-domain-library/` | the generic framework library the verbs open one section at a time |
| `skills/init/` | first-run setup, including the off-by-default local logging consent |
| `scripts/` | the mechanical gates: `preflight.sh`, `hostcheck.sh`, `tag-lint.sh`, `inference-gate.sh`, `publish-lint.sh`, `release.sh`, `telemetry.sh` |
| `config/` | `packs.json` (context registry) · `local.template.json` (every host-specific key, all null) · `dependencies.json` (per-verb capabilities) · `denylist.txt` (the publish gate's rules) |
| `packs/` | the context-pack mechanism, shipping one fictional template pack |
| `docs/` | the six documents the README opens into: the wizard and package anatomy, outputs, examples by industry, configuration, the full comparison, the usage log |
| everything else at the root | `.claude-plugin/plugin.json` (what a harness reads to install this), `config.schema.json`, `README.md`, `AGENTS.md`, `LICENSE`, `.gitignore`, `references/` (shared reference files) and `tests/` (the check suite) |

## The protocol (five gates, in order, every session)

Everything below this section is craft. These five are **preconditions**. Skipping one does not make
a run faster, it makes the output unfalsifiable.

**1. Config before advice.** Before you diagnose, recommend, or open a verb: resolve the context pack
(`config/packs.json`, a unique match on a company or alias selects it; no match is **generic mode**,
a legitimate answer) and resolve the machine-local config (`config/local.json`, copied from
`config/local.template.json`). Then run the routed verb's job-scoped preflight:

```bash
PKG_ROOT="${CLAUDE_PLUGIN_ROOT:-${PKG_ROOT:-}}"
[ -n "$PKG_ROOT" ] && [ -r "$PKG_ROOT/.claude-plugin/plugin.json" ] || { echo "STOP: set PKG_ROOT to this package's root directory, the one holding .claude-plugin/plugin.json, then re-run."; exit 2; }
bash "$PKG_ROOT/scripts/preflight.sh" --help          # what each verb needs, and its stack keys
bash "$PKG_ROOT/scripts/preflight.sh" <verb> --stack <keys>
```

An unset `${local:<key>}` is a **named miss**: say which key is unset and what filling it would
unlock, then continue on what you can prove. Never substitute a default, and never render another
operator's value or another company's name into this run's output. In generic mode and in every setup
flow, zero company literals appear anywhere in what you print. If `eng_plugin.name` is unset, the
plugin is in **standalone mode**, say so in one line and route standalone rather than offering a
handoff that has nowhere to go (`references/eng-handoff-adapter.md`).

**2. Locked context before work, and unanswered stays unanswered.** Echo the locked block back
(company/pack, area, output, destination, out-of-bounds) and wait for a confirmation or a correction
before routing. Every slot nobody answered travels forward **named as unanswered**, as a
`NEEDS-CONFIRMATION` row carrying its confirmer and what it blocks. An agent that quietly fills a
blank has replaced the operator's judgement with its own and hidden the swap.

**3. Evidence before belief.** Every factual claim carries exactly one of six tags: `FOUND`,
`INFERRED`, `CONSTRUCTED`, `CALCULATED`, `HYPOTHESIS`, `NEEDS-CONFIRMATION`. `FOUND` requires a
locator you can reopen (file and line, a URL, a section anchor, or a quoted snippet), a number
without a checkable source is not `FOUND`, whatever you believe about it. `HYPOTHESIS` never survives
into a delivery-final document; its only legal residues are the assumptions log and the open-questions
queue. The grammar lives in `skills/pm-requirements-v1/references/evidence-tags.md`, and it, not this
file, is the source of truth. Check it mechanically before you present anything:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naderelewa/Product-to-Prod](https://github.com/naderelewa/Product-to-Prod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
