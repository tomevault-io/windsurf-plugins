---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Suno Model Version

Target model: **Suno v5.5** (released 2026-03-26; Voices, Custom Models, My Taste): still current, no v6 released; checked 2026-07-31. When this stamp goes stale, re-verify and overwrite it in place, and keep it in sync with the "current as of" stamp near the top of `SKILL.md`.

## What This Repo Is

This is an **agent skill** (no application code, no build system). It packages instructions and reference material that teach an agent harness how to produce Suno v5.5-ready song packages: hook-first vocal lyrics, instrumental prompts/arrangements, vocal `Style`/`Excluded Styles` prompts, vocal/custom slider suggestions, and multilingual adaptations. The skill is harness-agnostic: anything that can read `SKILL.md` and selectively load `references/` can use it.

The git repo is named `suno-songwriter-agent-skill`; the working directory and skill name are both `suno-songwriter`. The skill name in `SKILL.md`'s frontmatter is canonical.

## Architecture

Progressive disclosure is the core design constraint:

- **`SKILL.md`**: the always-loaded entry point. Contains the YAML frontmatter (`name`, `description`) used for trigger matching, the core workflow, the output contract, and a routing table pointing to `references/`. Keep this file concise; deeper guidance belongs in a reference file.
- **`references/templates.md`**: the routing index. Every other reference file must be listed here. When you add a reference file, update both `templates.md` and the routing list near the top of `SKILL.md`.
- **`references/*.md`**: focused, independently-loadable topic files (output skeletons, Suno controls, Suno meta-tags, Suno sounds/samples, cover art/animated covers, studio/iteration, songwriting craft, song structure, rule-of-3, 80/20, instrumental, screen music, multilingual, style selection, genre palettes, genre fusion/multi-genre, remix/style-match, personas, albums, evals). A harness loads only the ones relevant to a request. Prefer adding a new narrow file over growing an existing one. Note: `personas.md` and `albums.md` instruct the agent to write spec `.md` files into the *user's* working directory (`personas/<slug>.md`, `albums/<slug>.md`), not into this repo.
- **`docs/`**: GitHub Pages documentation site, a single self-contained `docs/index.html` (hash-routed SPA, no build step) plus `.nojekyll`, served from `main`/`docs` at https://regiellis.github.io/suno-songwriter-agent-skill/. It summarizes `SKILL.md` and `references/`; it is not a second source of truth. When the output contract, a mode, or Suno controls change, update the matching page section. Design tokens follow the workspace `KARA.md` family (Fraunces/Source Serif 4, warm paper, `#8A3B12` accent); no em-dashes in prose.
- **`agents/openai.yaml`**: harness-specific metadata only. Portable behavior stays in `SKILL.md`/`references/`; product-specific config stays under `agents/`. Other harnesses get their own file (`agents/claude.yaml`, etc.); do not put harness-specific behavior in `SKILL.md`.

The skill's output contract is defined in `SKILL.md` and mirrored in `references/output-templates.md`: vocal songs use Title / Style / Excluded Styles / Lyrics, while instrumentals use Title / Instrumental Prompt / Arrangement. Optional Language Notes / Suno v5.5 Notes / Slider Suggestions / Checks apply only when useful and supported by the mode.

## Validation

There is no test runner. Validate changes with:

```bash
# YAML frontmatter / metadata parse check
ruby -e 'require "yaml"; YAML.load_file("SKILL.md"); YAML.load_file("agents/openai.yaml"); puts "YAML OK"'

# Compare source against the installed copy
diff -rq SKILL.md /Users/rellis/.agents/skills/suno-songwriter/SKILL.md
diff -rq references /Users/rellis/.agents/skills/suno-songwriter/references
```

`references/evals.md` holds the manual smoke-test matrix (should-trigger queries, should-not-trigger queries, regression checks, output rubric). Run those by hand when changing trigger wording or the output contract.

## Release Packaging

Users install via release zips (Claude Desktop upload, Codex `~/.codex/skills/`). To cut a release: promote CHANGELOG Unreleased to a version heading, commit, tag `vX.Y.Z`, then build and attach the zip:

```bash
# from a scratch dir: stage the folder, zip with the folder at the zip root
mkdir -p pkg/suno-songwriter
cp -R <repo>/SKILL.md <repo>/references <repo>/agents <repo>/README.md <repo>/CHANGELOG.md <repo>/LICENSE pkg/suno-songwriter/
(cd pkg && zip -r suno-songwriter.zip suno-songwriter)
gh release create vX.Y.Z pkg/suno-songwriter.zip --title "vX.Y.Z" --notes "<summary>"
```

The zip excludes `CLAUDE.md`, `KIKO.md`, `docs/`, and `.git`: repo-development files, not skill runtime. Never clobber a published release's zip with changed content; cut a patch release instead so the CHANGELOG and assets stay in step. Claude Desktop requires the folder (not bare files) at the zip root and reads name/description/license from `SKILL.md`.

## Installed Copy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [regiellis/suno-songwriter-agent-skill](https://github.com/regiellis/suno-songwriter-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
