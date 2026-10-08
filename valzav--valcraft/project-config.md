---
trigger: always_on
description: - `plugins/valcraft/` — the shipped plugin. Everything under it is copied into a consumer's plugin cache on install; everything outside it is development scaffold.
---

# Agent instructions

## Orientation

- `plugins/valcraft/` — the shipped plugin. Everything under it is copied into a consumer's plugin cache on install; everything outside it is development scaffold.
- `plugins/valcraft/skills/<skill>/SKILL.md` — one directory per skill, with its own `references/`, `templates/`, or `evals/` beside it.
- `plugins/valcraft/scripts/` — executables that ship with the plugin. `controller-stop.sh` is the Claude Code Stop hook.
- `docs/` — repository documentation.

Read the docs relevant to your change before modifying a skill or a manifest. Do not invent missing requirements — ask, or record the assumption in the change.

## Writing standard

Use these rules for documentation, plans, skill instructions, code comments, reviews, issues, commit messages, and PR text.

- Write for quick and unambiguous reading.
- Preserve precise terms, necessary qualifiers, and natural English.
- Prefer active voice when the actor matters.
- Keep each sentence focused. Split sentences that contain unrelated ideas or multiple instructions.
- Use one consistent term for each project concept. Do not change terms only for variety.
- Keep each paragraph focused on one topic.
- Use lists when prose would hide steps, options, or conditions.
- Define unfamiliar domain terms once. Keep established technical terms and necessary jargon.
- Avoid long noun chains, vague pronouns, and missing subjects.
- Preserve facts, conditions, exceptions, and scope. Never remove meaning only to make text shorter.
- Treat sentence length as a clarity signal, not a hard limit.
- Do not hard-wrap prose paragraphs, list items, or blockquotes. Keep each on one physical line and rely on soft wrapping. Preserve line breaks only when Markdown syntax or embedded content requires them, such as headings, tables, code blocks, frontmatter, HTML blocks, and explicit hard breaks.

For instructions, prompts, safety rules, and error messages:

- Put one action in each instruction.
- Name the actor when it is not clear.
- Prefer direct commands and simple sentence structures.

## Skill interaction design

- Minimize operator questions. Resolve a value from existing configuration, repository state, or history when one authoritative source exists, and apply it without asking. Ask only when the choice is genuinely open or authoritative sources conflict.
- Do not ask the operator to confirm answers they just gave. An interactive answer authorizes the write it configures; show the exact result in the report instead of a confirmation question.

## Commands

There is no application build or dependency install. The shipped plugin is Markdown, JSON, and YAML; standard-library Python scripts validate repository contracts and generated metadata.

- Develop a skill live: `claude --plugin-dir /path/to/valcraft/plugins/valcraft`, then `/reload-plugins` after each edit.
- Exercise the Cursor local-load path: `agent --plugin-dir /path/to/valcraft/plugins/valcraft`. The CLI has no plugin-reload command; reload the IDE window when a `--plugin-dir` session is not in use. Use a disposable Teams or Enterprise marketplace for install-path checks. Never mutate the operator's live Cursor configuration or team marketplace without explicit approval.
- Exercise the Codex packaging path: add the repository as a local marketplace with `codex plugin marketplace add /path/to/valcraft`, then install the cached copy with `codex plugin add valcraft@valcraft`. Use an isolated Codex profile for verification; never mutate the operator's live configuration without explicit approval.
- Validate the Codex plugin: Codex ships no standalone plugin validator, so the isolated-profile install above is the Codex manifest check. It must succeed and place the plugin in a cache directory named for the manifest version. Validate `plugins/valcraft/plugin.json` against `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json` with any JSON Schema validator.
- Run a skill's evals (`plugins/valcraft/skills/<skill>/evals/evals.json`): invoke the skill-creator skill with "run the evals for plugins/valcraft/skills/<skill>". Direct its workspace to `.local/` (gitignored) — never to the default sibling location inside `plugins/valcraft/`.
- Regenerate the OpenCode skills index after any change under `plugins/valcraft/skills/` outside `evals/`: `python3 scripts/build-skills-index.py` (CI runs it with `--check`). OpenCode consumes the skills through that index over raw GitHub; there is no OpenCode manifest.
- Check the federated worker-report registry, routing codes, backend returns, and active transport-deviation eval references: `python3 scripts/check-coordination-contracts.py`. Run its discriminating static tests with `python3 scripts/tests/test_check_coordination_contracts.py`. These checks detect declaration drift; behavioral evals prove behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valzav/valcraft](https://github.com/valzav/valcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
