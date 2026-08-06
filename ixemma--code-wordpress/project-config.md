---
trigger: always_on
description: This repository documents the 1st Providence WordPress and Elementor rebuild workflow.
---

# AGENTS.md

## Project Purpose

This repository documents the 1st Providence WordPress and Elementor rebuild workflow.

The public repository is a **learning blueprint**, not a full WordPress backup. It teaches followers how to:
- Plan WordPress page structures
- Build Elementor page templates with consistent styling
- Organize reusable components
- Document page blueprints and design systems
- Maintain workflow separation between public documentation and private outputs

## Public Repository Rules

1. **Public content** must never expose private client work, generated templates, databases, credentials, or production media.
2. **Documentation is public**: blueprints, style guides, section maps, and design notes.
3. **Placeholder files are public**: `.gitkeep` files and README files inside output folders.
4. **Reference exports are optional**: Approved public reference exports may be placed in `elementor-outputs/references/exports/` if the owner explicitly approves.
5. **Never commit generated Elementor templates** unless intentionally reviewed and approved for public use.

## Private Data Rules

**Never commit these file types:**

- `.env`, `.env.*`, `wp-config.php` (credentials)
- `elementor-outputs/elementor-templates/*.json` (generated templates, ignored by default)
- `elementor-outputs/private-templates/**` (private templates, always ignored)
- `*.sql`, `*.sql.gz`, `*.wpress`, database files (backups)
- `wp-content/`, `wordpress/`, `uploads/`, `backup/`, `database/` folders
- `*.jpg`, `*.png`, `*.webp` (client media, ignored by default)
- `AGENTS.local.md`, `AGENT_PRIVATE_NOTES.md`, `LOCAL_DAILY_WORK.md` (local notes, ignored)

## Required Folder Structure

```
codex-skills/                              # Public skills and instructions
docs/
  1ST_PROVIDENCE_BLUEPRINT.md              # Main blueprint
  ELEMENTOR_BUILD_NOTES.md                 # Build guidance
  STYLE_GUIDE.md                           # Design system
  PERSONAL_CARE_AIDE_PCA_BLUEPRINT.md      # Program blueprint
  BASIC_LIFE_SUPPORT_PROVIDER_BLUEPRINT.md # Program blueprint
  REPO_STRUCTURE_AND_OUTPUT_WORKFLOW.md    # Output workflow (public)
elementor-outputs/
  README.md                                # Output folder guide (public)
  elementor-templates/                     # Generated templates folder
    .gitkeep                               # Placeholder (public)
    README.md                              # Naming guide (public)
  private-templates/                       # Private templates folder
    .gitkeep                               # Placeholder (public)
    README.md                              # Rules (public)
  references/
    README.md                              # Reference guide (public)
    exports/
      .gitkeep                             # Placeholder (public)
    screenshots/
      .gitkeep                             # Placeholder (public)
  section-maps/                            # Section maps folder
    .gitkeep                               # Placeholder (public)
README.md                                  # Main project README
PROJECT_PROGRESS.md                        # Public milestones only
IMPLEMENTATION_PHASES.md                   # Workflow phases
BUGS_AND_SOLUTIONS.md                      # Known issues and solutions
AGENTS.md                                  # This file (public)
.gitignore                                 # Scoped ignore rules
```

## Elementor Output Rules

### Generated Elementor Templates

**Location:** `elementor-outputs/elementor-templates/`

**Format:** Elementor JSON `.json` files

**Git behavior:** Ignored by default. Never commit without owner approval.

**Naming format:**
```
page-name-page-body-template.json
page-name-section-name-template.json
page-name-component-name-template.json
```

**Examples:**
```
homepage-body-template.json
nurse-aide-page-body-template.json
medication-management-page-body-template.json
ekg-why-students-choose-section-template.json
header-mega-menu-template.json
```

**Important:** When a task asks you to "build a page," "build a section," or "build a template," you must produce a real Elementor `.json` file. Markdown-only output is not sufficient.

### Private Templates

**Location:** `elementor-outputs/private-templates/`

**Git behavior:** Completely ignored. Never commit from this folder.

**Use cases:**
- Client-sensitive page layouts
- Production-only templates
- Early draft versions
- Day-to-day working exports

## Reference Export Rules

**Location:** `elementor-outputs/references/exports/`

**Git behavior:** Allowed in Git. Must be explicitly approved before committing.

**When to use:**
- Public-safe Elementor references
- Approved sample exports
- Learning materials

**When NOT to use:**
- Private client layouts
- Production-sensitive exports
- Unreleased designs

## Daily Work Privacy Rules

1. **Day-to-day work notes stay local.** Never commit daily worklog files.
2. **Use local ignored files for private notes:**
   - `AGENTS.local.md` for local agent instructions
   - `AGENT_PRIVATE_NOTES.md` for private development notes
   - `LOCAL_DAILY_WORK.md` for daily work tracking
3. **Commit only public milestones** to `PROJECT_PROGRESS.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ixEmma/code-wordpress](https://github.com/ixEmma/code-wordpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
