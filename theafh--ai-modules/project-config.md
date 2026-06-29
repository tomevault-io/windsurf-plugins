---
trigger: always_on
description: **ai-modules is a meta-repository.** It defines AI components — skills, agents, commands, hooks — and packages them as plugins. Treat every `SKILL.md`, `plugin.json`, and `marketplace.json` as a published artefact.
---

# AGENTS.md

**ai-modules is a meta-repository.** It defines AI components — skills, agents, commands, hooks — and packages them as plugins. Treat every `SKILL.md`, `plugin.json`, and `marketplace.json` as a published artefact.

## What this repo is not

The shipped skills are the **product**, not the workflow. When a user asks you to apply one while editing this repo, confirm whether they mean to invoke it or edit its definition.

## Layout

```text
.agents/plugins/marketplace.json # Codex marketplace registration
.claude-plugin/marketplace.json  # Claude marketplace registration
plugins/<plugin>/
  .codex-plugin/plugin.json       # Codex plugin metadata (uses "skills": "./skills/")
  .claude-plugin/plugin.json      # Claude plugin metadata
  README.md                       # plugin overview + skill list
  skills/<skill>/SKILL.md         # skill definition with YAML frontmatter
deployment/                       # deploy script + per-tool config
tests/                            # local-only regression harnesses (gitignored)
Makefile                          # task entry point
.markdownlint.jsonc               # markdown lint config (MD033 off — pseudo-XML is intentional)
```

## Authoring conventions

- **Use pseudo-XML inside skill prompts** (`<role>`, `<objective>`, `<policy>`, `<output_contract>`). Reference: `plugins/ai_dev/skills/ai_instruction_formatting/SKILL.md`.
- **Use positive, action-oriented language** in skill prose and instructions. Reference: `plugins/ai_dev/skills/ai_instruction_writing/SKILL.md`.
- **Write skill descriptions for both audiences.** The `description:` frontmatter is read before the body is loaded by an LLM router and by users browsing skills. Serve both needs deliberately: give the user a precise compact summary of what the skill is about and how it differs from neighbors, then give the router keyword-rich `Use when` trigger contexts, prompt phrases, artefacts, file types, and invocation boundaries. Keep implementation workflow details in the body.
- **Keep the toolchain to Make + shell + markdown.** Add new languages, package managers, or build steps only when the user explicitly asks for them.
- **Match snake_case naming** for skill and plugin directories.
- **Name skills and agents by invocation mode and collision risk.** A skill that is the only entry point for its capability keeps the ordinary family-first name, even when it delegates to agents (`wiki_fix`). Use `<family>_auto_<rest>` for an agent-delegating automation skill when it needs to sit beside a classical/manual skill with the same capability or subset (`task_auto_implement` beside `task_implement`). A spawned agent leads with `auto_` and ends with the family token (`auto_<role>_<family>`, e.g. `auto_implementer_task` or `auto_shaper_wiki`) and is not intended to be invoked by the user.
- **Write deployment-agnostic cross-references.** Reference sibling artefacts by name (`auto_shaper_wiki`, `format_markdown`) rather than by plugin name, marketplace, or installed path.

## Versioning

- **Ship a new skill, agent, or plugin at 1.0.0.** In the commit that first introduces it, leave the version at 1.0.0 — no bump.
- **Bump once per commit, with the change — and only at commit time.** When a commit edits an existing skill, agent, or plugin, raise its `version` in that commit. Do not bump while iterating, and do not add version-bump steps to task files, plans, or pre-commit notes.
- **Use patch increments for minor maintenance changes.** For a small follow-up, wording fix, or environment-specific hint, advance only the patch component.
- **Plugin meta stays lockstep.** When a skill or agent `version:` rises, raise the matching plugin's `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`, and marketplace entries to the same new plugin version in the same commit. Adding a skill to an existing plugin counts as a plugin edit; the new skill itself ships at 1.0.0.
- **Local marketplaces stay current for `ai_dev` and `knowledge_management`.** When a commit publishes a new version of either plugin, update the local marketplace registrations in `.agents/plugins/marketplace.json` and `.claude-plugin/marketplace.json` in the same commit as the plugin manifests. When opening this repo for plugin maintenance or after updating either marketplace registration, compare the installed cached skill versions against the repo source and tell the user to restart Codex or open a new thread so the refreshed plugin cache is loaded.

## Common tasks

- `make help` — list every target.
- `make lint` / `make fix` — runs `markdownlint`, `jq` syntax check, `shellcheck`. `fix` auto-fixes markdown only.
- `make deploy` — symlink components into vendor config dirs. Aliases: `global`, `install`. **Run only when the user asks for it.**
- `make uninstall` — remove deployed artefacts via the deployment log.

## Protected file edits

- **Treat `.agents/plugins/marketplace.json` as protected when Codex runs with managed filesystem permissions.** Read it for context, then attempt at most one exact-context `apply_patch` for the intended JSON field.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theafh/ai-modules](https://github.com/theafh/ai-modules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
