---
trigger: always_on
description: This repository is a curated skills pack for AI coding agents — skills sharing one naming system, one tone, one structure. Your job is to maintain that consistency when adding or editing skills.
---

# AGENTS.md — skills-by-yigitkonur

This repository is a curated skills pack for AI coding agents — skills sharing one naming system, one tone, one structure. Your job is to maintain that consistency when adding or editing skills.

## What this repo is

A single combined skills pack — not a loose collection. Every skill must feel like it belongs to the same family. Consistency, clarity, and install-path stability are more important than clever naming or one-off structure.

This repo is a `skills` CLI pack, a Claude Code plugin marketplace, and a Codex plugin marketplace. Codex consumes the complete `skills/` folder; Claude Code receives explicit allowlists that exclude entries in `CODEX_ONLY_SKILLS`. The `-secondary` b-side repo was merged in and archived; never point anything back at it.

**Distribution model:**
- Claude Code plugin marketplace: `/plugin marketplace add yigitkonur/skills-by-yigitkonur`, then `/plugin install <skill>@yigitkonur`, a bundle `yk-*@yigitkonur`, or `yk-everything@yigitkonur`. Codex-only skills are absent from all four Claude surfaces.
- Codex plugin marketplace: `codex plugin marketplace add yigitkonur/skills-by-yigitkonur`, then install `<skill>@yigitkonur` or the all-pack `skills-by-yigitkonur@yigitkonur` from `/plugins`.
- `skills` CLI full pack: `npx -y skills add -y -g yigitkonur/skills-by-yigitkonur`
- `skills` CLI single skill: `npx -y skills add -y -g yigitkonur/skills-by-yigitkonur/skills/<skill-name>`

The plugin metadata is **generated** from `skills/` by `scripts/gen-marketplace.py`:
- `.claude-plugin/marketplace.json`: Claude-compatible per-skill plugins + themed `yk-*` bundles + `yk-everything` + `yk-researchers`, all `source: "./"` + `strict: false` + explicit `skills` allowlists so Codex-only skills cannot leak through a broad directory reference.
- `.codex-plugin/plugin.json`: the root Codex plugin manifest for the all-pack plugin.
- `plugins/<skill>/`: one generated, self-contained Codex plugin package per canonical skill.
- `.agents/plugins/marketplace.json`: the Codex repo marketplace entries for the all-pack and every individual skill.

Regenerate plugin metadata whenever you add, remove, or rename a skill. Place a Claude-compatible skill in exactly one `GROUPS` bundle; place a runtime-specific Codex skill in `CODEX_ONLY_SKILLS` and no Claude bundle.

**Agents:** subagent suites live in `subagents/` — deliberately NOT the conventional `agents/` name, because every plugin uses `source: "./"` and Claude Code auto-discovers an `agents/` folder at the plugin root, which would attach the agents to *every* installed skill. Marketplace entries reference explicit agent-file lists (`RESEARCHER_AGENTS` / `BROWSER_AGENTS` in `gen-marketplace.py`, never a bare folder): the internet-researcher suite ships with `yk-researchers`, `yk-research`, and `yk-everything`; the agent-browser tester/extractor suite ships with `yk-automation`, the `run-agent-browser` per-skill plugin, and `yk-everything`. A new agent file must be added to the matching list (or a new one) in `gen-marketplace.py` or it ships nowhere. Never rename `subagents/` back to `agents/`.

**Versioning:** `VERSION` is the single source of truth; `gen-marketplace.py` stamps it onto every plugin entry, and `.github/workflows/version-bump.yml` patch-bumps it only after an explicit confirmed manual dispatch; routine pushes do nothing.

## Repository layout

```
.
├── skills/                         # All skills live here
│   └── <verb>-<object>/            # Each skill directory
│       ├── SKILL.md                # Required — the skill definition (hand-written)
│       ├── README.md              # Required — per-skill install instructions
│       ├── references/             # Optional — deep-dive docs routed from SKILL.md
│       ├── scripts/                # Optional — helper scripts paired with docs
│       └── assets/                 # Optional — templates or fixtures
├── scripts/
│   ├── validate-skills.py          # Validates all skills (references, frontmatter, junk)
│   ├── gen-marketplace.py          # Generates .claude-plugin/marketplace.json from skills/
│   └── bump-version.py             # Bumps VERSION patch + regenerates marketplace (CI)
├── VERSION                         # Single source of truth for plugin versions (CI-bumped)
├── subagents/                      # Subagent suites (NOT auto-discovered)
│   ├── claude/                     # Claude Code variants — researcher + agent-browser suites, shipped via explicit agents lists
│   └── codex/                      # Codex variants (researchers only) — for ~/.codex/agents, not the marketplace
├── .agents/plugins/
│   └── marketplace.json            # Generated — Codex plugin marketplace catalog (do not hand-edit)
├── .claude-plugin/
│   └── marketplace.json            # Generated — plugin marketplace catalog (do not hand-edit)
├── .codex-plugin/
│   └── plugin.json                 # Generated — Codex all-pack plugin manifest (do not hand-edit)
├── plugins/
│   └── <skill>/                    # Generated — self-contained Codex plugin packages (do not hand-edit)
├── .github/workflows/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yigitkonur/skills-by-yigitkonur](https://github.com/yigitkonur/skills-by-yigitkonur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
