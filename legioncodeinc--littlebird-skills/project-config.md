---
trigger: always_on
description: This is the always-on briefing for any coding agent working in this repository. Claude
---

# Littlebird Skills project guidance

This is the always-on briefing for any coding agent working in this repository. Claude
Code, Cursor, Codex, and Cowork all read this file, directly or through a thin import.

## What this repo is

A Claude plugin marketplace of skills built on the Littlebird MCP. Littlebird captures
screen activity and transcribes meetings, so the record of the user's work already
exists. Every skill here turns some part of that record into something actionable.

Thirty skills live under `skills/`, in six groups:

- **Money and business operations.** money-leak-auditor, renewal-sentinel,
  invoice-chaser, deal-pipeline-reconstructor.
- **Lead generation and growth.** lead-harvester, comment-to-crm-piper,
  content-repurposer, said-it-already, testimonial-miner, competitor-watch.
- **Meetings and follow-through.** meeting-scribe, commitment-tracker,
  who-am-i-ghosting, pre-call-prep, client-health-radar.
- **Personal productivity.** daily-brief, day-reconstructor, focus-forensics,
  learning-capturer, weekly-review.
- **Knowledge and writing.** sop-forge, knowledge-base-builder, osint-investigator,
  research-synthesizer, brand-voice-guardian.
- **Meta and automation.** routine-architect, skill-suggester.
- **Voice.** combined-voice-creator, littlebird-voice-creator, facebook-voice-creator.
  These predate the current structure and do not carry a research archive.

The repo root is simultaneously the plugin (`.claude-plugin/plugin.json`, skills
auto-discovered from `skills/`) and the marketplace (`.claude-plugin/marketplace.json`
listing the plugin with source `./`). Keep both manifests in sync when anything is added
or renamed.

**One plugin, deliberately.** Installed plugins are copied to a cache directory and
cannot reference files outside themselves, so splitting into several plugins would break
the cross-skill references that exist today. If a split is ever revisited, the blockers
are: content-repurposer reads thirteen paths under `said-it-already/references/`, and
every skill README links siblings at `../<skill-name>/README.md`.

## Operating rules

1. Do not add em dashes or en dashes to authored prose anywhere in this repo. Use
   ordinary punctuation and the spaced hyphen " - " for asides. This repo exists to kill
   AI tells; its own files don't get to have them. Preserve literal data and verbatim
   source material exactly as captured. Check the actual characters before committing,
   do not eyeball it.
2. Protect user work. Never discard unrelated changes, rewrite history, or delete broad
   paths without clear authorization and a verified target.
3. Raw personal data never ships. Facebook exports, Littlebird retrievals, and any user
   corpus are working material only. They get processed in temp space and deleted. Only
   distilled, user-confirmed content lands in a skill.
4. Every fact a skill encodes about a person, a company, a commitment, or a number gets
   confirmed with the user first. Unconfirmed facts do not ship, ever.
5. Skill frontmatter uses only the Agent Skills spec fields: `name`, `description`,
   `license`, `compatibility`, `metadata`, `allowed-tools`. Anything else fails a
   claude.ai or Cowork upload. Nest `version`, `author`, and `requires` inside `metadata`.

## The skill contract

Every skill under `skills/` follows this shape:

```
skills/<skill-name>/
├── SKILL.md          Instructions for the model. Spec-six frontmatter.
├── README.md         The page a human reads to decide. No frontmatter.
├── references/
│   ├── <guides>.md                   Domain guides, one per major verb
│   ├── littlebird-mcp-reference.md   Shared, duplicated across skills
│   ├── evidence-standards.md         Shared, duplicated across skills
│   └── research/
│       ├── README.md
│       ├── distilled-<topic>.md      Cited distillation
│       └── raw/                      One file per archived source
└── scripts/          Only where a deterministic helper earns its place
```

`SKILL.md` carries these sections in order: Purpose, Littlebird MCP calls used, Trigger,
Routine cadence, Process, Output, Guardrail, Related skills. The folder name and the
frontmatter `name` must match exactly, or Cursor's discovery breaks silently.

**Shared references are duplicated, not linked.** `littlebird-mcp-reference.md` and
`evidence-standards.md` are copied verbatim into every skill that needs them, because a
copied plugin cannot reach outside its own directory. If you edit one, update ALL copies
in the same change. The same rule already applied to the voice skills' Facebook guides
and `voice-skill-template.md`.

## The evidence standards (do not dilute them)

These are why the marketplace is trustworthy. They are set out in full in each skill's
`references/evidence-standards.md`.

1. **Receipts or it did not happen.** Every claim from capture carries source, app, and
   timestamp.
2. **Observed, inferred, external, and unknown are four different things**, and the
   difference is visible to the reader. Never promote an inference by dropping the hedge.
   Never turn an absence into a negative finding.
3. **The attribution guardrail.** Capture shows what the user was viewing, not what they

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [legioncodeinc/littlebird-skills](https://github.com/legioncodeinc/littlebird-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
