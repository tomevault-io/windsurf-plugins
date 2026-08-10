---
trigger: always_on
description: This skill is indexed at [skills.sh/agentsorg/design-engineering/design-engineering](https://www.skills.sh/agentsorg/design-engineering/design-engineering). Discovery is automatic — skills.sh re-indexes on `npx skills add` calls. If the live page shows stale metadata after a release, re-run `npx skills add AgentsORG/design-engineering` once to trigger a re-index. There is no manual publish step.
---

# AGENTS.md

Repo-level operating guidance for AI agents working in this codebase. Pairs with [SOUL.md](SOUL.md), which carries identity (voice, stance, taste lineage). This file is the **what / how**; SOUL.md is the **who / why**. Read both on cold start.

## What this repo is

A single agent skill packaged as a **skill graph** — one root `SKILL.md` (the Map of Content) plus a `references/` folder of atomic, wikilinked nodes organised into themed subfolders (`philosophy/`, `motion/`, `typography/`, `surface/`, `components/`, `layout/`, `anti-patterns/`, `meta/`) and an `evals/` folder for Perplexity-style Step-0 evals. A sibling `agents/` directory ships six workflow subagents that map to the highest-value design-engineering tasks. The skill encodes design engineering knowledge.

Since v2.0.0 the repo is organised around **four primitives**: Knowledge (`skills/design-engineering/`, Agent Skills spec), Package (root `plugin.json`, [Agent Plugins v1.0.0](https://agent-plugins.org/)), Runtime (`agent/` + `evals/`, the [eve](https://eve.dev/) framework), and Client extensions (`agents/`, `commands/`, and the per-host manifest dirs, declared under reverse-domain namespaces in `plugin.json` `extensions`). The knowledge is canonical; everything else delivers it. `agent/skills/` is **generated** by `scripts/sync-skills.mjs` — never edit it; edit `skills/design-engineering/` and re-sync.

The graph is intentionally **Obsidian-compatible**: installers can open this repo as a vault to navigate clusters in the graph view and edit nodes in place. The recommended companion is [kepano/obsidian-skills](https://www.skills.sh/kepano/obsidian-skills). Preserving that compatibility is a constraint on every edit (see rule 6).

## When editing this repo

**1. Edit atomic nodes, not the SKILL.md.** The SKILL.md is a map. Real content lives in `references/<theme>/`. If you're adding new knowledge, drop the atomic node into the matching themed subfolder and link it from that theme's `MOC-<theme>.md`. Only update SKILL.md when adding a new top-level topic (which means adding a whole new theme folder).

**2. Atomic nodes are short and standalone.** Target 40–80 lines per node. Each node:
- States the principle in the first paragraph.
- Backs it with a source (Emil, Benji, guidelines.sh, or HKTITAN).
- Ends with a gotcha or counter-example.
- Links to neighbors via `[[node-name]]` (no `.md` extension).

**3. The description in SKILL.md frontmatter is a routing trigger.** It says *when* to load the skill, not *what* the skill does. Phrasing should match real user queries ("designing a component", "why does this feel flat", "should this animate"). Do not rewrite it without an eval.

**4. Gotchas append, instructions don't grow.** When the agent gets something wrong, add a one-liner to `references/gotchas.md`. Do not lengthen existing nodes or rewrite the description.

**5. Wikilinks carry meaning.** Embed `[[name]]` in prose, not in a bare "See also" list at the end. Per Akshay's framing: the link itself is an instruction to the model about when/why to follow it.

**6. Keep the graph Obsidian-compatible.** Installers view and edit this skill as an Obsidian vault. That means: wikilinks stay as `[[name]]` (no path, no `.md` extension), every node carries YAML frontmatter so the graph view can index it, and filenames stay unique across the vault — Obsidian resolves `[[name]]` by basename, so two files sharing a name silently collide. Never introduce path-prefixed wikilinks (`[[references/name]]`) or `.md`-suffixed ones; both break the bare-name pattern and degrade the graph view.

## File conventions

- Filenames: lowercase, hyphenated, no extension in wikilinks, **unique across the vault** (Obsidian resolves wikilinks by basename, so theme subfolders are organizational only — they don't create namespaces).
- Theme folders: `references/<theme>/` — one per cluster. Current themes: philosophy, motion, typography, surface, components, layout, anti-patterns, meta.
- MOCs: `references/<theme>/MOC-<theme>.md` — one per theme folder. Indexes that theme's atomic nodes.
- Atomic nodes: `references/<theme>/<concept-name>.md`.
- A new theme = a new folder + a new `MOC-<theme>.md` + a link from SKILL.md. Don't create themes for fewer than 3 nodes — fold into an existing one.
- Frontmatter on every node (light: `title`, `summary`, `tags`) — required for Obsidian graph indexing.
- **Subagents**: `agents/<agent-name>.md` — YAML frontmatter (`name`, `description`, `tools`, `model`) plus a system-prompt body. Hoisted to repo root for `npx plugins` / Cursor plugin discovery; names match the basename so wikilinks resolve. Don't add a subagent unless it has a workflow the main agent can't do cheaply inline.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentsORG/design-engineering](https://github.com/AgentsORG/design-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
