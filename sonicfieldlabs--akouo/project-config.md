---
trigger: always_on
description: AKOÚŌ is a multimodal listening system and portable skill library for AI agents. It gives agents accountable, switchable, operational ears for analyzing sound, audio, music, and sonic concepts across thirteen distinct listening modes, one meta-router, and one conceptual reference layer.
---

# AGENTS.md — AKOÚŌ

## What This Project Is

AKOÚŌ is a multimodal listening system and portable skill library for AI agents. It gives agents accountable, switchable, operational ears for analyzing sound, audio, music, and sonic concepts across thirteen distinct listening modes, one meta-router, and one conceptual reference layer.

It is NOT a generic audio-analysis tool. It is a framework for *how* agents should listen, what each mode reveals, what it hides, and what must remain unknown.

## Core Principles

1. **Claim Taxonomy is sacred.** Every output must separate findings into:
   - `heard` — directly present in the input
   - `measured` — produced by technical inspection
   - `inferred` — plausible logical deductions only (not theory/culture)
   - `interpreted` — cultural, theoretical, affective, or contextual readings
   - `speculative` — declared fictional or imaginative readings
   - `undetermined` — what cannot be responsibly claimed

2. **Listening modes are mutually corrective.** No single mode has the final word.

3. **The router decides.** `akouo-router` analyzes the listening situation and assigns primary, secondary, and corrective modes.

4. **Agents do not hear like humans.** The system does not pretend otherwise.

5. **Public release hygiene governs all edits.** Do not add private paths, unpublished research directories, personal data, local notes, credentials, private recordings, or non-public source maps to the portable release.

## Repository Structure

```
akouo/
  skills/              # Portable agent skills, one per folder
    <skill>/           # 13 listening modes + akouo-router + reference-layer
      SKILL.md         # Skill instructions with YAML frontmatter
      references/      # Bundled JSON schemas for strict output
  schemas/             # Canonical JSON schemas (shared source of truth)
  commands/            # Command definitions that chain skills
  examples/            # Example outputs and reference maps
  app/                 # Local-first reference app (Vite + React, no model provider required)
  scripts/
    validate-release.sh  # Pre-release validation
  evals/               # Evaluation checklists and notes
  README.md
  SYSTEM_GUIDE.md      # Operational guide: commands, workflows, benchmark notes
  SKILL_INDEX.md       # Quick-reference manifest of all skills
  AGENTS.md
  LICENSE
  .gitignore
```

## Skill Format

Each skill must follow this structure:

```
skill-name/
  SKILL.md             # Required. YAML frontmatter + markdown body
  references/          # Required for AKOÚŌ core skills; enables standalone use
    *.schema.json
```

### SKILL.md requirements

- **YAML frontmatter** at the very top:
  - `name`: kebab-case identifier matching the folder name
  - `description`: A "pushy" paragraph telling agents WHEN to trigger this skill. Include concrete use cases. Combat undertriggering by being explicit and generous with triggering conditions.
  - `compatibility`: Brief note about agent framework support and schema dependencies.
- **Body**: Markdown sections for Purpose, When To Use, Core Question, Input Assumptions, Listening Procedure, Output Structure, Guardrails, Recommended Next Modes, and Examples.
- **Output fields**: Must document every field of the shared listening output.
- **Schema references**: Point to `references/*.schema.json` for standalone portability. The canonical schemas live in `schemas/`.

### Schema bundling rules

- Every listening mode bundles `listening-output.schema.json` and `claim-taxonomy.schema.json` in `references/`.
- The router bundles `router-output.schema.json`, `routing-plan.schema.json`, `listening-output.schema.json`, and `claim-taxonomy.schema.json`.
- The reference layer bundles `reference-map.schema.json`, `listening-output.schema.json`, and `claim-taxonomy.schema.json`.
- Do NOT modify schema contents when copying. Keep them identical to `schemas/`.
- If you update a canonical schema in `schemas/`, you MUST re-copy it to all affected `references/` folders (`./scripts/validate-release.sh` verifies this).

## When Modifying Skills

1. **Preserve the name field.** Do not change skill identifiers unless you are renaming the entire mode.
2. **Keep descriptions pushy.** The description is the primary trigger mechanism. If a skill is not being invoked when it should, expand the description with more triggering conditions.
3. **Do not add personal data.** No names, addresses, API keys, private recordings, or local paths.
4. **Maintain epistemic consistency.** The claim taxonomy (`heard`/`measured`/`inferred`/`interpreted`/`speculative`/`undetermined`) must be respected uniformly across all skills.
5. **Update examples if you change behavior.** Examples are the fastest way for agents to understand a skill.
6. **Update the README** if you add, remove, or rename skills or commands.
7. **Keep conceptual refinements portable.** Public theory and generic terminology are fine; private project names, local provenance, and local file paths are not.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sonicfieldlabs/akouo](https://github.com/sonicfieldlabs/akouo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
