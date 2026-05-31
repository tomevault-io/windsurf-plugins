---
trigger: always_on
description: > **Single source of truth.** This file is loaded by Claude (`CLAUDE.md` → `@AGENTS.md`), Codex (`AGENTS.md` directly), and Gemini (`GEMINI.md` → `@AGENTS.md`). All three tools share this context. Edit here; never edit `CLAUDE.md` or `GEMINI.md` directly.
---

# Responsibility Laundering Book — Project Instructions

> **Single source of truth.** This file is loaded by Claude (`CLAUDE.md` → `@AGENTS.md`), Codex (`AGENTS.md` directly), and Gemini (`GEMINI.md` → `@AGENTS.md`). All three tools share this context. Edit here; never edit `CLAUDE.md` or `GEMINI.md` directly.

Project: **No One Did It: Responsibility Laundering, from the Scapegoat to the Algorithm**.

Use this workspace to research, structure, draft, audit, and market a serious trade nonfiction book about responsibility laundering.

## Global Five Over-Rules

<!-- GENERATED:five-over-rules:start -->
1. **Evidence before elegance.** Never improve the story by weakening the evidence.
2. **Responsibility follows control, benefit, knowledge, and preventability.** Do not stop at the most visible actor.
3. **Keep the taxonomy intact.** Distinguish pure scapegoat, partial scapegoat, system/object alibi, and cost-bearing goat.
4. **Steelman before judgment.** Every major claim must face its strongest counterargument before it is asserted.
5. **Handoff cleanly.** Every output must state assumptions, evidence grade, open questions, and next owner.
<!-- GENERATED:five-over-rules:end -->

## Core book thesis

Civilization did not stop sacrificing substitutes. It changed the altar. Responsibility laundering is the recurring method by which power keeps control or benefit while moving blame, liability, moral cost, or public suspicion onto a weaker bearer, procedural shell, proxy, machine, market, committee, legal entity, or record.

## Core diagnostic

For every case ask:

1. Who or what was publicly blamed?
2. Who had control?
3. Who benefited?
4. Who knew or should have known?
5. Who could have prevented recurrence?
6. Who controlled the record?
7. Who bore the cost?
8. What would responsibility look like if it followed control instead of visibility?

## Case taxonomy

- **Pure scapegoat:** mostly innocent actor is blamed to close the case.
- **Partial scapegoat:** actor is involved or guilty at one level, but blame stops too low and protects a wider chain.
- **System/object alibi:** machine, model, benchmark, corporation, committee, market, legal form, or procedure becomes the blame container.
- **Cost-bearing goat:** victim or worker is not accused as guilty but absorbs harm while responsibility is avoided.

## Style rules

- Use narrative heat for scenes and analytical coldness for conclusions.
- Do not invent scenes, dialogue, motives, or thoughts.
- Do not make all cases morally identical.
- Do not create partisan symmetry or false equivalence.
- Do not let novelty outrank verifiability.
- Prefer official records, court files, inquiries, watchdog reports, primary documents, and high-quality investigative sources.
- Flag every live/contested case as such.

## Model policy

**All 13 crew agents run on `opus`.** Principal Author directive: logic-laden tasks require the larger model. The crew does no purely-mechanical work — every role involves multi-step reasoning, judgment under constraints, or reconciliation of contradictory inputs. Sonnet is reserved for ad-hoc utility tasks outside the crew (e.g., one-shot prose tweaks where logic is not load-bearing).

Verified at every test run by the per-agent `model:` frontmatter check; any drift to sonnet on a crew agent should fail review.

## Default artifact standard

Every substantial output should include:

```text
Owner:
Purpose:
Evidence grade:
Assumptions:
Open questions:
Handoff:
```

## Repository layout

All inputs and outputs referenced by the crew now live inside this project
directory (`books/responsibility_laundering/`). The previous artifact-bundle
ZIPs were extracted and relocated under `dev-docs/`; the active crew
workspace was lifted from `dev-docs/03_CLAUDE_AI_CREW_WORKSPACE/extracted/responsibility-laundering-book-ai-crew/`
to the project root.

### Active workspace (project root)

- `AGENTS.md` — this file; the single source of truth for all three tools.
- `CLAUDE.md` / `GEMINI.md` — one-line `@AGENTS.md` imports; do not edit directly.
- `README.md` — crew workspace quick-start.
- `.claude/agents/` — the 13-agent AI crew (Jerry, Bonnie, Wayne, Delon, Shirley, Selina, Warren, Loki, Stephen, Laura, Nancy, Alan, Blair). See `.claude/docs/crew-portfolio.md` for the full roster.
- `.claude/skills/` — 14 project-owned skills (`case-file-method`, `counter-case-method`, `chapter-blueprint`, `responsibility-chain-mapping`, `source-ledger-discipline`, `citation-hygiene`, `counterargument-red-team`, `publication-proposal`, `evidence-grading`, `taxonomy-classification`, `defamation-wording`, `scene-construction`, `primary-source-playbooks`, `vocabulary`) plus the `.claude/skills/cc-suite/` symlink (→ `~/.claude/plugins/cache/xiaolai/cc-suite/<ver>/skills/cc-suite`) installed by `/cc-suite:bridge-skills` so Codex sees plugin-provided sub-skills via `.agents/skills`. The plugin symlink is bridge infrastructure, not a project skill, and is excluded from NLPM scans (no top-level `SKILL.md`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaolai/no-one-did-it](https://github.com/xiaolai/no-one-did-it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
