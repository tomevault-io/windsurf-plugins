---
trigger: always_on
description: This is the GTM Skills plugin repo. These guidelines are for AI assistants working on skills and plugin code in this repository.
---

# Claude Instructions — GTM Plugin Development

This is the GTM Skills plugin repo. These guidelines are for AI assistants working on skills and plugin code in this repository.

## Repo Structure

```
gtm-plugin/
├── .claude-plugin/              # Root plugin manifest
│   ├── plugin.json
│   └── marketplace.json
├── .github/
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.yml       # YAML form template for skill bug reports
│       └── config.yml           # Issue template chooser config
├── AGENTS.md                    # This file — dev guidelines (CLAUDE.md symlinks here)
├── ALL_SKILLS.md                # Auto-generated skill listing
├── skills/                      # All skills in one flat directory
│   ├── bootstrap/
│   │   └── SKILL.md
│   ├── seo-and-aeo-strategy/
│   │   └── SKILL.md
│   ├── linkedin-content/
│   │   └── SKILL.md
│   └── ... (56 skills total)
├── scripts/                     # Build and validation scripts
│   ├── bump-version.sh          # Bump semver across all manifests
│   └── generate-skills-readme.sh # Regenerate ALL_SKILLS.md + README skills section
└── validate-skills.sh           # Skill spec validator
```

Each skill follows this layout:

```
skills/<skill-name>/
├── SKILL.md             # Skill definition (YAML frontmatter + markdown)
├── references/          # Supporting reference material
├── assets/              # Templates, examples
└── rules/               # (optional) Organized guidelines
```

## User Project Structure (the `/bootstrap` convention)

When bootstrap runs in a user's project, it creates this structure. **All skills must reference these paths consistently.**

```
user-project/
├── CLAUDE.md                    # AI instructions: repo structure, naming, workflow, routing
├── about/
│   └── me.md                    # Personal voice, writing style, personality, biography
├── strategy/
│   └── brand.md                 # Brand positioning, messaging, audience, competitors, voice
├── content/
│   ├── ideas.md                 # Content idea backlog with status
│   ├── calendar.md              # Publishing schedule
│   ├── <platform>/
│   │   ├── drafts/              # Work-in-progress content
│   │   └── published/           # Archive of published content
│   └── ...                      # linkedin/, twitter/, reddit/, blog/, email/
└── assets/
    ├── logos/                   # Logo files
    └── brand/                   # Brand visual references
```

### What each folder is for

| Path | Purpose | Who writes to it |
|---|---|---|
| `CLAUDE.md` | AI instructions — repo structure, what to read first, naming, workflow | Bootstrap creates it; rarely updated |
| `about/` | Personal and team context | User provides; skills read before voice-matched content |
| `strategy/` | Brand foundation — positioning, messaging, voice, audience, competitors | Bootstrap creates; updated when brand evolves |
| `content/` | Day-to-day content engine — ideas, planning, drafts, published | Skills create drafts; user publishes |
| `assets/` | Reusable visual assets — logos, brand files | User provides |

### Content naming convention

All content files (drafts and published) use:

```
YYYY-MM-DD_short-topic-slug.md
```

### Content lifecycle

1. **Idea** → add a line to `content/ideas.md`
2. **Draft** → create file in `content/<platform>/drafts/`
3. **Publish** → move file from `drafts/` to `published/`
4. **Repurpose** → new draft in target platform's `drafts/`, referencing original

## Writing Skills — Conventions

### Reading user context

When a skill needs brand or personal context, use these paths. Check if they exist before reading:

```
# Brand context (strategy, positioning, voice)
strategy/brand.md

# Personal voice context
about/me.md

# Alternate legacy paths (check as fallback)
.claude/product-marketing-context.md
.agents/product-marketing-context.md
```

**Pattern for skills:**
```markdown
Check for `strategy/brand.md` first. If it exists, read it before asking questions.
Use that context and only ask for information not already covered.
```

### Generating content

Skills that produce draft content must:

1. Save to `content/<platform>/drafts/` (not root, not `artifacts/`)
2. Use the `YYYY-MM-DD_short-topic-slug.md` naming convention
3. Never write generated content into `strategy/`, `about/`, or `CLAUDE.md`

### Routing new learnings

When a skill discovers new information during execution:

- Brand shifts, new messaging, audience insights → `strategy/brand.md`
- Writing preferences, voice discoveries → `about/me.md`
- New content ideas → `content/ideas.md`

### Referencing internal skill files

Skills can bundle their own reference material in `references/`, `assets/`, or `rules/` subdirectories. Reference them with relative paths:

```markdown
See [references/templates.md](references/templates.md) for examples.
```

### Bootstrap skill

Bootstrap is a skill at `skills/bootstrap/SKILL.md`. It must:

1. Generate the same unified folder structure (see above)
2. Create `CLAUDE.md`, `about/me.md`, `strategy/brand.md` as foundational files
3. Create all content platform folders with `.gitkeep` in empty leaf directories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manojbajaj95/claude-gtm-plugin](https://github.com/manojbajaj95/claude-gtm-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
