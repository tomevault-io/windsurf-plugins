---
trigger: always_on
description: This repo is a collection of agent skills. Each skill is one self-contained folder under `skills/<bucket>/<slug>/` with a `SKILL.md` inside. Skills work with any coding agent that supports the [skills.sh](https://skills.sh) installer (Claude Code, Cursor, Codex, Cline, Windsurf, OpenCode).
---

# Skills Repo Conventions

This repo is a collection of agent skills. Each skill is one self-contained folder under `skills/<bucket>/<slug>/` with a `SKILL.md` inside. Skills work with any coding agent that supports the [skills.sh](https://skills.sh) installer (Claude Code, Cursor, Codex, Cline, Windsurf, OpenCode).

## Layout

```
skills/
  engineering/
    <slug>/SKILL.md
  productivity/
    <slug>/SKILL.md
```

Buckets in use:

- `engineering/` for code-adjacent practice (audits, scaffolds, refactors)
- `productivity/` for process and discipline (planning, writing, design)

The two folders are the on-disk split (operates on a codebase vs operates on a process). They do not affect discovery: the agent auto-invokes a skill by reading its `description`, and skills install by slug. So the folder is just a maintenance home. The root `README.md` browse view groups skills by **domain** instead (Frontend/UI/UX, Game, AI & agents, Writing & workflow, Mobile, Security), which is how a human actually looks for one, and each `SKILL.md` carries a `tags:` facet (see the format below) for finer slicing. At ~16 skills this shallow tree is deliberate. Do not add domain folders until a cluster reaches roughly five skills sharing a tag; grow a folder from an earned tag cluster, not a guess.

## Adding a skill

When you add a new skill, touch five places. Forgetting any of them silently degrades the install experience or leaves the skill untested for invocation.

1. `skills/<bucket>/<slug>/SKILL.md` plus `skills/<bucket>/<slug>/permissions.yaml`. The skill itself. Every skill folder carries a `permissions.yaml` capability manifest (the card's `bom` counts it, so a missing one desyncs the card). For a pure-knowledge skill it is the five-line block used by `no-slop` and `autopilot`. `model: knowledge`, `executes: false`, `network: none`, `shell: false`, `filesystem_writes: false`. Those stay `false` even when the skill's guidance has the agent write files, because that is the agent's capability, not the shipped artifact's.
2. `README.md`. Add a line under the matching **domain group** in the Skill reference section (the listing groups by domain, not by bucket). Link the slug to its SKILL.md.
3. `skills/<bucket>/README.md`. Add a line in the bucket index.
4. `.claude-plugin/plugin.json`. Add `./skills/<bucket>/<slug>` to the `skills` array.
5. `evals/cases/<slug>.yaml`. Invocation eval cases: a few `should_fire` prompts, a couple `should_not_fire`, and `route_to_sibling` cases for any skill it overlaps with. See [evals/README.md](evals/README.md).

Then run `pnpm check` (which runs `node scripts/check-skills.mjs`) and `pnpm eval:validate`. The first enforces every skill rule below (the `: ` and ` #` frontmatter traps, `name` matching the folder, the 1024-char description cap, plugin.json registration, that every relative link inside the skill resolves, and that any `references/` directory is a conformant OKF bundle). The second checks the invocation cases are valid YAML, match the schema, and cover every skill. Both exit non-zero on any failure. Run them before you commit.

The `javascript-ecosystem` skill is a dated snapshot of a fast-moving ecosystem, so it carries extra anti-staleness machinery: a snapshot `date`, a per-notes-file `**Verified YYYY-MM-DD**` stamp, a freshness section in its `SKILL.md`, and `skills/engineering/javascript-ecosystem/MAINTENANCE.md`. Run `node scripts/check-freshness.mjs` to list the oldest entries due for a re-verify against official docs.

## Trust cards (re-signing a changed skill)

Most skills carry a `CARD.md` (a `trust-card` OKF concept binding the skill's content digest, signature, and capability manifest) plus a rendered `CARD.svg`. `cards.json` at the repo root is the aggregate render feed. `pnpm cards:check` gates CI: it rebuilds every card and fails if any is stale, so a skill whose files changed must have its card regenerated, rebuilt, and re-signed before merge.

The lifecycle when you edit a skill's content, in order (finish all content edits first, since the digest is computed over the bundle):

1. **Regenerate the card** so its `target_digest` matches the new content:
   ```sh
   python3 skills/engineering/trust-card/scripts/card.py generate skills/<bucket>/<slug> \
       --identity did:web:saschb2b.com --expires <YYYY-MM-DD>
   ```
   This rewrites `<slug>/CARD.md` and `<slug>/CARD.manifest.json`. Convention for `--expires` is roughly one year out (the existing cards were minted with a 2027-06-29 horizon).
2. **Rebuild the render feed** for that skill: `pnpm cards <slug>` (updates `cards.json` and `<slug>/CARD.svg`). Run bare `pnpm cards` to rebuild all.
3. **Sign the bound digest.** This is Sascha's interactive step and cannot be done from an agent session:
   ```sh
   python3 skills/engineering/trust-card/scripts/card.py sign skills/<bucket>/<slug>/CARD.md
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saschb2b/skills](https://github.com/saschb2b/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
