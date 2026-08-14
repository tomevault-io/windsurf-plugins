---
trigger: always_on
description: `jstack` is a Markdown repo for Jesus-centered software resources, Christian AI standards, agent skills, and LLM prompts.
---

# AGENTS.md

## Project Overview

`jstack` is a Markdown repo for Jesus-centered software resources, Christian AI standards, agent skills, and LLM prompts.

There is no app runtime, package manifest, build system, test runner, database, or CI workflow.

Key areas:

- `README.md` - human-facing overview and entry points.
- `prompts/` - copy-ready Christian AI prompts.
- `skills/` - installable Skills CLI content, including `ft-build-christian-ai-guardrails`, `ft-evaluate-christian-ai-apps`, `ft-find-bible-developer-resources`, `ft-once-keep-agents-and-readme-fresh`, and `ft-remove-ai-code-slop`.
- `resources/` - standalone teachings and reference docs.

## Repository Structure

```text
.
+-- README.md
+-- CONTEXT.md
+-- docs/
|   +-- adr/
|       +-- 0001-synergize-as-spectrum.md
+-- prompts/
|   +-- christian-discipleship-ai-prompt.md
|   +-- christian-discipleship-ai-prompt-compact.md
+-- resources/
|   +-- AM_I_OPEN_HANDED.md
|   +-- FAITH_TOOLS_AI_STANDARDS.md
|   +-- WHAT_TO_DO_WITH_AN_APP_IDEA.md
+-- skills/
    +-- christian-ai-creator-helper/                (archived)
    |   +-- SKILL.md
    |   +-- references/
    +-- ft-build-christian-ai-guardrails/           (archived)
    |   +-- SKILL.md
    |   +-- references/
    +-- ft-discern-christian-app-idea/
    |   +-- SKILL.md
    |   +-- references/
    +-- ft-evaluate-christian-ai-apps/
    |   +-- SKILL.md
    |   +-- references/
    +-- ft-find-bible-developer-resources/
    |   +-- SKILL.md
    |   +-- references/
    +-- ft-once-keep-agents-and-readme-fresh/
    |   +-- SKILL.md
    +-- ft-remove-ai-code-slop/
        +-- SKILL.md
```

## Setup Commands

No install is needed for normal work.

- Install skills locally: `npx skills add cameronapak/jstack`
- Check status before edits: `git status --short`

Do not add package managers, lockfiles, build tools, formatters, or test frameworks unless asked.

## Development Workflow

- Edit Markdown directly.
- Keep human-facing docs in `README.md`.
- Keep agent guidance in `AGENTS.md`.
- Put reusable prompts in `prompts/`.
- Put installable skill content in `skills/<skill-name>/SKILL.md`.
- Every `skills/<skill-name>/SKILL.md` must include Agent Skills `metadata` with at least `fruit` set to exactly one of the nine qualities in Galatians 5:22–23, as a lowercase string: `love`, `joy`, `peace`, `patience`, `kindness`, `goodness`, `faithfulness`, `gentleness`, `self-control`. Pick one that fits the skill’s intent; do not list the fruit in `README.md` unless asked.
- Put skill support files in `skills/<skill-name>/references/`.
- Put broader resources in `resources/`.
- Preserve filenames and links unless asked to rename them.

## Documentation Freshness

Repo reality is the source of truth. If `AGENTS.md` or `README.md` becomes false, update it in the same change when the fix is objective.

Objective facts include repo structure, tracked paths, setup commands, validation commands, runtime/tooling, skill/resource/prompt inventory, workflow constraints proven by the repo, and each skill’s `metadata.fruit` (Galatians 5:22–23) in frontmatter.

- Update `AGENTS.md` when it is stale about agent-facing repo reality.
- Update `README.md` when it is stale about human-facing purpose, entry points, install, or use.
- Ask before changing policy, philosophy, positioning, or workflow intent.
- If both docs are stale, update both. Do not make them mirror each other unless the same fact belongs in both.
- Ignore temporary, generated, local-only, and unrelated untracked files.
- If unrelated user changes make docs look stale, ask before broadening scope.
- After repo-reality changes, check `AGENTS.md` and `README.md` before finishing.
- In the final response, mention any freshness updates.

## Validation Commands

This repo is content-only. Validate structure and links.

- Show changed files: `git status --short`
- Review Markdown diffs: `git diff -- '*.md'`
- Check compact prompt size: `wc -m prompts/christian-discipleship-ai-prompt-compact.md`

Keep the compact prompt under `8000` characters unless asked otherwise. A recent route.bible target was under `7500` characters.

When adding or editing prompt links, verify referenced files exist. Use relative links from the source document.

If editing untracked Markdown, `git diff -- '*.md'` will not show it. Read the file or use an explicit no-index diff before summarizing.

After skill add/rename, confirm every skill has `metadata.fruit` (line `  fruit:` immediately under `metadata:` in frontmatter):

```bash
for f in skills/*/SKILL.md; do grep -q '^  fruit:' "$f" || echo "missing metadata.fruit: $f"; done
```

Expect no output. If a file is missing it, add `fruit` per the Development Workflow list.

## Testing Instructions

There is no automated test suite.

For prompt and guardrail changes, verify:

- Scripture accuracy rules remain.
- The prompt does not fabricate Bible quotes.
- AI identity stays clear and non-human.
- Prayer guardrails remain explicit. AI must not say "I'll pray for you" or "let's pray together." It may offer words the user can pray.
- Human community referrals remain for prayer, confession, doctrine, loneliness, pastoral care, crisis, abuse, and discipleship.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cameronapak/jstack](https://github.com/cameronapak/jstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
