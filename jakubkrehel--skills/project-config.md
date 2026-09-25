---
trigger: always_on
description: This file is the single source of guidance for coding agents working in this repository. `CLAUDE.md` imports it and adds nothing but Claude Code specifics, so put repository facts here and do not maintain a second copy.
---

# AGENTS.md

This file is the single source of guidance for coding agents working in this repository. `CLAUDE.md` imports it and adds nothing but Claude Code specifics, so put repository facts here and do not maintain a second copy.

## What this repository is

A collection of agent skills for building great product interfaces (typography, colors, UI polish), distributed two ways: via `npx skills add jakubkrehel/skills` and as the Claude Code plugin `interfaces` served by the marketplace in this same repository. It is documentation-only; there is no build, lint, or test tooling.

`.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` define the plugin and its marketplace. Both are named `interfaces`, so plugin users invoke skills as `/interfaces:better-interface` while skills-CLI users invoke `/better-interface`. Skills are discovered from `skills/` automatically, so adding a skill needs no manifest change. Bump `version` in `plugin.json` in the same commit as any change under `skills/`. That number is the only signal plugin users update on. `claude plugin update` compares it and nothing else, so a change shipped without a bump reports "already at the latest version" and never reaches them. Run `claude plugin validate .` and `claude plugin validate .claude-plugin/plugin.json` after touching either manifest.

`opencode.json` registers `skills/` under `skills.paths` so opencode loads the collection while this repository itself is open, which is for working on the skills rather than distributing them. opencode users install through the skills CLI's opencode target, and opencode exposes every discovered skill as a slash command on its own, so this repository carries no opencode command wrappers.

## Structure

Each skill lives in `skills/<skill-name>/`, with `SKILL.md` as the entry point and supporting `.md` files beside it.

Skills come in two shapes. A domain skill holds knowledge: what is true about typography, color, or layout. A verb skill holds a procedure: review this change, explore these variants, explain this interface. A procedure sitting inside a domain skill is a candidate for extraction, and a domain rule sitting inside a verb skill belongs to its owner instead.

The content headings belong to the skill, not to a house style: a set of files all filling one section template reads like instances of one file. What is shared is the small amount of framing that calibrates behaviour rather than organising content.

Every `SKILL.md` carries:

- **Frontmatter** with `name` (matching the directory) and `description`.
- **A plain-name H1** and a two-sentence opener saying what the skill is and what it does. Not what the domain means or why it matters: an agent does not need motivating, and a reader can tell the difference.
- **A calibration line or two**, in the opener or in its own section where it needs the room. This is where a skill says how hard to press: which values are exact rather than approximate, what counts as a finding versus a preference, when the right answer is to write nothing. A skill that lists rules without saying how hard to press leaves that to chance. That is the difference between a review that blocks on evidence and one that blocks on taste. Give the section a heading that carries its own point (`Evidence, not taste`), not a generic label.
- **Headings that carry the point**, in sentence case. `Native elements first`, not `Semantics`. Number them only where the steps genuinely run in order, as `better-interface` and `interface-review` do; numbering flat reference implies a sequence that isn't there and makes every insertion a renumber.
- **A hand-off line** naming the sibling skills that own adjacent topics.
- **A `## Before you finish` table**, two columns, where the domain has recurring mistakes. The left column is the detection pattern, which is what a principle statement does not give you. The heading names the moment on purpose: `Common mistakes` is a label an agent reads past while orienting, and `Before you finish` names the point in the work where the table is worth consulting.
- **A `## Reporting` section** in every domain skill, carrying that domain's severity ladder, its verification checks and the format for a standalone review. See below.

Supporting `.md` files carry depth beyond the principle statements: recipes, code patterns, lookup tables. Link each one from the principle that needs it, so the link sits where the agent lands. A principle states the rule and links out for the recipe. It never restates the reference file in shorter form, and the reference file never restates the principle in longer form.

Each rule lives in exactly one skill. Other skills point to it by skill name in backticks (`better-layout`), never by cross-skill relative link, because each skill directory ships on its own.

Point at a principle by its heading in bold (**Classify every finding**), never by its number. A numbered reference breaks silently the moment a principle is inserted above it, and nothing in the file fails when it does.

### The review format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakubkrehel/skills](https://github.com/jakubkrehel/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
