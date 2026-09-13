---
trigger: always_on
description: Conventions for any AI agent (Claude Code, Codex, Cursor, Copilot, …) editing this repository.
---

# AGENTS.md

Conventions for any AI agent (Claude Code, Codex, Cursor, Copilot, …) editing this repository.

## What this repo is

`vinta-ai-workflows` — a private npm package that ships **bootstrap skills** (Agent Skills format) into other projects' AI-tooling directories (`.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`). The package itself contains:

- `vinta-ai-workflows.mjs` — single-file Node CLI (`install` / `update` / `uninstall` / `list`). Dependency-free. Node ≥ 18.
- `skills/<name>/SKILL.md` — the `vinta-`-prefixed bootstrap skills users invoke after installing.
- `skills/vinta-derive-skills/resources/foundation-skills/<name>/` — **foundation-skill templates** that get copied verbatim into target projects' `ai-tools/skills/` at bootstrap time. These are *content shipped to other repos*, not source compiled here.
- `skills/vinta-derive-skills/resources/plan-execution/` — the **plan-execution unit**: thin shell templates (`shell/{implement-plan,implement-phase,review-phase,integrate-phase,amend-plan}-template.md`) that `<!-- include -->` shared fragments from `partials/` (implementer prompt, model pick, review layers, worktree/`WORKROOT` seam, PR-context, commit-strategy bodies). `vinta-derive-skills` expands includes → substitutes `{{…}}` → writes one `ai-tools/skills/<name>/SKILL.md` per shell. See [plan-execution/README.md](skills/vinta-derive-skills/resources/plan-execution/README.md).
- `skills/vinta-derive-skills/resources/systematic-debugging-template.md` — the other placeholder-rendered skill body (opt-in).
- `skills/vinta-bootstrap-ai-tools/resources/stacks/<stack>/notes.md` — per-stack detection signals + skill / agent categories. **Notes only — no ready-made content.**
- `schemas/*.v1.schema.json` — JSON Schema Draft 2020-12 definitions for every YAML payload the skills produce or consume. `schemas/README.md` documents versioning.
- `scripts/*.mjs` — **source-side maintenance scripts** run by CI, never shipped (excluded from the `files` whitelist). `check-ai-models.mjs` is the nightly freshness check for the `plan-feature` AI model tier table (`resources/ai-models.yaml`): it checks the cited ids against a **free, no-key model aggregator** (models.dev, with LiteLLM's JSON as fallback) — detection needs network access but no API keys — and on drift (a cited id disappeared, or a newer same-family model shipped) has an LLM propose an updated table that `.github/workflows/check-ai-models.yml` opens as a PR. The LLM proposal is the only step that wants a key (`ANTHROPIC_API_KEY`), and it's optional. May use devDependencies (e.g. `yaml`) — this does **not** weaken the CLI's zero-runtime-deps property, which only concerns `dependencies` + `vinta-ai-workflows.mjs`.
- `CHANGELOG.md` — Keep a Changelog format, SemVer.

The repo is **self-recursive**: it authors skills it itself does not run. Don't try to "test" a skill by invoking it inside this repo — invoke it inside a target project after `npx vinta-ai-workflows install`.

### `dev-skills/` — maintenance skills for this repo

Separate from `skills/` (which ships to consumer projects), the top-level `dev-skills/` directory holds skills agents load **when editing this repo itself**. They are NOT shipped to consumers — `dev-skills/` is excluded from the package via `package.json`'s `files` whitelist and from the CLI's `SKILLS_SRC` discovery (which only walks `skills/`).

Vendor auto-discovery is wired via committed symlinks at the repo root — no per-developer setup step:

```
.claude/skills                  → ../dev-skills   (Claude Code)
.cursor/skills                  → ../dev-skills   (Cursor)
.github/skills                  → ../dev-skills   (VS Code + Copilot)
.agents/skills                  → ../dev-skills   (Codex; also picked up by Cursor + Copilot)
.github/copilot-instructions.md → ../AGENTS.md    (Copilot reads this file)
```

`.gitignore` un-ignores exactly these paths past the otherwise-ignored vendor dirs (`.claude/*`, `.cursor/*`, `.agents/*`). New committers don't run anything — `git clone` + open the editor is enough. To add a new dev skill: drop the dir under `dev-skills/<name>/` and commit. The symlinks expose it automatically.

| Skill | When to use |
|---|---|
| [add-foundation-skill](dev-skills/add-foundation-skill/SKILL.md) | Add a new foundation skill under `skills/vinta-derive-skills/resources/foundation-skills/<name>/`. Walks the schema-ripple checklist (schema enum, bootstrap interview, derive-skills bucket, foundation-shape lists, outputs tree, CHANGELOG). |
| [add-stack](dev-skills/add-stack/SKILL.md) | Add a new stack template (`skills/vinta-bootstrap-ai-tools/resources/stacks/<stack>/notes.md`). Wires through the orchestrator stack table + `vinta-analyze-codebase` detection signals. Notes-only — never bundles ready-made content. |
| [release](dev-skills/release/SKILL.md) | Cut a release. Pre-flight checks (clean tree, on `main`, fetched), version bump, CHANGELOG section close, commit + tag + push. Surfaces the publish command — never auto-publishes. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vintasoftware/vinta-ai-workflows](https://github.com/vintasoftware/vinta-ai-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
