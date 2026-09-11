---
trigger: always_on
description: > Org overview: [../../AGENTS.md](../../AGENTS.md)
---

> Org overview: [../../AGENTS.md](../../AGENTS.md)

# AGENTS.md

This repository contains **opencode skills and global agents** — Markdown-based instruction files installed into `~/.config/opencode/`. There is no application to run, no tests to execute, and no CI pipeline.

## Repository Structure

```
skills/          — 34 skill directories (each: SKILL.md + optional scripts/, references/, assets/)
agents/          — 11 agent definition files (YAML-frontmatter Markdown, *.md) + LICENSE.txt
.opencode/       — Local opencode config (opencode.json, prompts/, command/)
```

## Key Facts

### Skills are Markdown files, not code

Each skill is a `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`). The agent reads this file and follows its instructions. Bundled resources live alongside: `scripts/` (Python), `references/` (docs), `assets/` (templates).

**Editing a skill means editing its SKILL.md and any bundled files.** There is no compilation or build step.

### install.sh / install.ps1 is the deployment mechanism

`./install.sh` (Linux/macOS) or `./install.ps1` (Windows) copies skills, agents, MCP servers, commands, and config to `~/.config/opencode/`. It also removes stale skills/agents/MCP servers/commands no longer in the repo, installs Python deps via `uv`, replaces the global `opencode.json`, and copies prompt files. **After editing files in this repo, run the install script to apply changes to the active opencode configuration.**

Both installers must stay in sync. A new deployable directory added to one is a bug in the other.

### Python dependencies are managed by `uv`

`pyproject.toml` lists dependencies (pdfplumber, pypdf, pypdfium2, pillow, playwright, etc.). The virtual environment is installed at `~/.local/opencode-venv/` by the install script. Before running any Python scripts:

```bash
source ~/.local/opencode-venv/bin/activate
```

### Agent files use YAML frontmatter

Each `agents/<name>.md` has frontmatter with `description`, `mode: subagent`, and optionally `permission` and `model` fields. The body is Markdown instructions. The `description` triggers the agent — keep it specific and actionable. `agents/LICENSE.txt` is not an agent file.

### .opencode/ contains local config

`.opencode/opencode.json` configures providers, models, MCP servers, agent prompts, and the `instructions` files that carry memory. It references `prompts/plan.txt` and `prompts/build.txt` for the plan and build workflows.

### Prompt overrides are general-purpose by design

`.opencode/prompts/{build,plan,compaction}.txt` are **custom prompts**, wired via `agent.<name>.prompt` in `opencode.json`. They replaced opencode's defaults, which were coding-specific.

**The governing rule: these prompts carry methodology, not domain knowledge.** How to work — task management, evidence before assertion, scope discipline, delegation, communication. Domain technique belongs in skills. That split only holds because each prompt opens with a **Skills Come First** section instructing the agent to check for an applicable skill and to let it govern; removing that section reopens the gap that stripping the coding guidance created. Anything added here that is specific to code, or to any one domain, is in the wrong file.

- **`build.txt` and `plan.txt` deliberately diverge.** build is execution methodology; plan is investigation methodology (explore → constraints → options → agreement) with a definition of what a plan must contain. Upstream has no separate plan prompt — plan mode is enforced by injected `<system-reminder>` blocks and by permissions — so this divergence is ours, and both files still defer enforcement to those reminders rather than restating prohibitions.
- **Task Management is load-bearing, not boilerplate.** The upstream variant these replaced (`Ta`, the generic fallback) ships with *no* task-management section, which is why the agents did not maintain todo lists. Both prompts now specify: list before starting, one `in_progress`, complete immediately without batching, discovered work becomes new items, re-read before claiming done.
- **`compaction.txt` cooperates with a contract it cannot override.** opencode supplies its own user-side prompt demanding a fixed five-section structure wrapped in `<summary>` tags. The system prompt cannot change that, so it adds *fidelity* rules inside it: standing user instructions and unreconstructable literals (keys, IDs, paths, versions, exact commands) are copied verbatim, and narrative is what gets cut when space runs short. Do not add competing section structures here — they will fight the user prompt.
- **Overriding bypasses model-family selection.** opencode picks a default system prompt by substring-matching the model id (`claude`, `gpt`/`codex`, `gemini-`, `kimi`, `trinity`, `muse-spark`), falling through to a generic variant — which is what the ThreatWinds `qwen-*`/`silas-*` ids resolve to. With an override set, that selection no longer happens at all, for any model.
- **A prompt override replaces the model-family prompt entirely**, but not everything: the merge is `agent.prompt ? [agent.prompt] : provider(model)`, and the environment block plus any user system prompt are still appended after it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osmontero/opencode-skills](https://github.com/osmontero/opencode-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
