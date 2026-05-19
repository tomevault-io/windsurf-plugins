---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **For End Users:** See [README.md](README.md) for installation and usage.
>
> **This file** is for contributors, maintainers, and skill developers.

## What This Is

A **hybrid multi-plugin marketplace** for AI coding agents. Each plugin is
executable documentation (a Claude Code / Agent Skills compatible skill) loaded
on demand. Plugins are unrelated and versioned independently.

The manifest at `.claude-plugin/marketplace.json` lists every plugin. A plugin
is either:

- **External** — referenced via a GitHub `source` object. Content, tests, and
  releases live in that plugin's own repo. This repo only pins a ref.
- **Inline** — content lives here under `plugins/<name>/` and uses this repo's
  per-plugin release pipeline.

## Repository Structure

```
agent-plugins/
├── .claude-plugin/marketplace.json   # Marketplace + plugin entries
├── plugins/                          # Inline plugins only (empty until added)
│   └── .gitkeep
│   └── <inline-plugin>/              # (when added) source dir for the manifest
│       ├── skills/<plugin>/          # Autodiscovered: skills/<name>/SKILL.md
│       │   ├── SKILL.md              # Core skill file
│       │   └── references/           # Reference files loaded on demand
│       ├── tests/                    # Baseline scenarios, rationalization table
│       └── CHANGELOG.md              # Per-plugin changelog (CI-managed)
└── .github/workflows/
    ├── validate.yml                  # PR validation (hybrid source-aware)
    └── automated-release.yml         # Per-plugin release (inline plugins only)
```

Claude Code autodiscovers `<source>/skills/<name>/SKILL.md` and, for external
sources, auto-clones + caches the referenced repo (see
[plugins reference](https://code.claude.com/docs/en/plugins-reference)).

## Plugin Source Types

| | External | Inline |
|--|----------|--------|
| `source` | `{ "source": "github", "repo": "owner/repo", "ref": "vX.Y.Z" }` | `"./plugins/<plugin>"` |
| Content | Upstream repo | `plugins/<plugin>/` here |
| Releases | Upstream repo's own pipeline | This repo's per-plugin pipeline |
| Update here | Bump `source.ref` + mirrored `version` | Push a scoped conventional commit |
| `version` field | Optional, manual mirror of `source.ref` (NOT CI-managed) | Required, CI-managed, must equal SKILL.md `metadata.version` |

`terraform-skill` is external: `antonbabenko/terraform-skill`, pinned by
`source.ref`. Its content and tags (`vX.Y.Z`) live in that repo. Pins are
bumped automatically: the scheduled `Update External Plugins` workflow
(`.github/workflows/update-external-plugins.yml`) auto-discovers external
entries from `.claude-plugin/marketplace.json`, resolves the latest upstream
release, and opens a reviewable `chore(external-plugins): ...` PR updating
`source.ref` in both manifests plus the mirrored `version`. Per-plugin
overrides live in `.github/external-plugin-updates.json`; `validate.yml`
cross-checks the two manifests stay in sync. Do not hand-bump.

## Adding a Plugin

**External plugin** — manifest entry only:

1. Add to `.claude-plugin/marketplace.json` `plugins[]`: `name`,
   `source: { "source": "github", "repo": "owner/repo", "ref": "vX.Y.Z" }`,
   `description`, optional `category` / `keywords`, optional `version`
   (mirror of the ref, manual).
2. No local content, CHANGELOG, or tests here. No scoped-commit release.

**Inline plugin** — content lives here:

1. Create `plugins/<plugin>/skills/<plugin>/SKILL.md` with valid frontmatter
   (`name`, `description`; optional `metadata.version`).
2. Add a `plugins[]` entry: `name`, `source: ./plugins/<plugin>`,
   `description`, `version` (start at `0.1.0`), optional `category` /
   `keywords`.
3. Add `plugins/<plugin>/CHANGELOG.md` (can be empty; CI prepends to it).
4. The manifest `version` must equal the SKILL.md `metadata.version`. If the
   plugin ships a `.codex-plugin/plugin.json`, its `version` must match too.
   CI enforces this.
5. Add `plugins/<plugin>/tests/baseline-scenarios.md` - **required**, CI
   enforces it: at least one `## Scenario`, a `## Running These Tests`
   protocol, and a `### Success Criteria` list. Copy the shape of
   `plugins/code-intelligence/tests/baseline-scenarios.md`.

## Development Workflow

**This is documentation, not code.** No build, no compiled tests.

### Validation

CI runs on PRs touching `plugins/**` or `.claude-plugin/**`. It validates every
`plugins/*/skills/*/SKILL.md`. To check locally:

```bash
# Frontmatter + size, all skills
for f in plugins/*/skills/*/SKILL.md; do
  echo "$f: $(wc -l < "$f") lines"
done

# Manifest <-> SKILL.md version sync
python3 -c "
import json, yaml, os
m = json.load(open('.claude-plugin/marketplace.json'))
for p in m['plugins']:
    src = p['source'].lstrip('./')
    sp = os.path.join(src, 'skills', p['name'], 'SKILL.md')
    fm = yaml.safe_load(open(sp).read().split('---', 2)[1])
    sv = (fm.get('metadata') or {}).get('version')
    print(p['name'], p['version'], '==', sv, 'OK' if sv == p['version'] else 'MISMATCH')
"

# Broken internal links for a skill
cd plugins/<plugin>/skills/<plugin>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonbabenko/agent-plugins](https://github.com/antonbabenko/agent-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
