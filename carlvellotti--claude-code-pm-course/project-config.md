---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a dual-purpose repository for the Claude Code for Product Managers course:

1. **course-materials/** - Student-facing interactive course content (distributed via GitHub Releases as a zip)
2. **website/** - Next.js documentation site (deployed to Vercel at ccforpms.com)

Students download course-materials/ as a zip, extract it, run `claude` from that folder, and type `/start-1-1` to begin learning.

## Course Architecture: Config-Driven System

**This course uses a config-driven architecture for maximum flexibility when adding/reordering modules.**

### Single Source of Truth: course-structure.json

All module definitions live in `course-materials/course-structure.json`. This file controls:
- Course structure and module sequence
- Slash command routing (all slash commands are identical - they read the config to know which teaching script to load)
- Website navigation (generated at build time from the config)
- Teaching script navigation (modules read config to know what comes next)

### How It Works

**Slash Commands:**
- All 10 slash commands (`/start-1-1` through `/start-2-3`) are identical
- They parse their own command name (e.g., "start-1-2" → module "1.2")
- They read `course-structure.json` to find the module's teaching script path
- They load and execute that teaching script

**Teaching Scripts:**
- At the END of each module, scripts read `course-structure.json`
- They dynamically determine what comes next
- They tell students the correct slash command for the next module

**Website Navigation:**
- `website/pages/fundamentals/_meta.ts` and `website/pages/advanced/_meta.ts` import the config
- Navigation is generated at build time from `course-structure.json`

### Adding or Reordering Modules

To add a new module or reorder existing ones:

1. **Edit `course-materials/course-structure.json`** - Add/move module definition
2. **Create the new module folder and files** (if adding a new module)
3. **Done!** Everything else updates automatically:
   - Slash commands route correctly (they're all identical)
   - Teaching scripts reference the correct "next" module (they read the config)
   - Website navigation updates (generated from config at build time)

**No need to:**
- ❌ Rename folders
- ❌ Update individual slash command files
- ❌ Edit existing teaching scripts
- ❌ Update website `_meta.ts` files manually

**Example: Insert module 1.4 between current 1.3 and 1.4:**

Just edit `course-structure.json` to add the new module definition, and everything cascades automatically. The folder for the old 1.4 (agents) can stay named `1.4-agents` - the config maps logical IDs (1.5) to physical paths (1.4-agents).

### Benefits

- ✅ Add modules without touching existing files
- ✅ Reorder modules by editing one JSON file
- ✅ Website and course materials stay in sync automatically
- ✅ One source of truth for course structure
- ✅ Maximum flexibility for course evolution

## Critical: When Opening This Repository

**DO NOT proactively set up, build, or install anything** when the user first opens this repository unless explicitly asked. The README.md contains specific warnings about this:

- ❌ Do NOT run `npm install`
- ❌ Do NOT run `npm run build`
- ❌ Do NOT make setup changes
- ✅ Wait for explicit user instructions

This is an interactive course repository. The user (or students) will guide what needs to be done.

## Common Commands

### Release Management Workflow

**When You Update Course Content:**

1. **Make your changes** to files in course-materials/
   - Edit modules in `course-materials/lesson-modules/`
   - Update company context in `course-materials/company-context/`
   - Modify agents in `course-materials/.claude/agents/`
   - etc.

2. **Commit and push to main:**
   ```bash
   git add -A
   git commit -m "Update Module 1.3 with new examples"
   git push origin main
   ```

3. **Create a new release:**
   ```bash
   # Run the release script with new version number
   ./scripts/create-release.sh v1.0.1

   # Create GitHub release with the new zip
   gh release create v1.0.1 releases/complete-course.zip \
     --title "v1.0.1 - Updated Module 1.3" \
     --notes "- Fixed typos in Module 1.3\n- Added new examples to Module 2.1"
   ```

4. **Update the website (if needed):**
   - The homepage shows "Latest: v1.0.0"
   - Update this in `website/pages/index.mdx` line 128:
   ```markdown
   **👉 [Download Course Materials](...latest/download/complete-course.zip)** - Get the complete course (Latest: v1.0.1)
   ```

**How GitHub Releases Works:**
- `/releases/latest/download/complete-course.zip` - Always points to the most recent release
- Students using this URL automatically get the latest version
- You can create as many releases as you want (v1.0.1, v1.0.2, v1.1.0, etc.)
- Old releases stay available at their specific version URLs

**Semantic Versioning Guide:**
- `v1.0.X` (Patch) - Bug fixes, typos, minor updates to existing content
- `v1.X.0` (Minor) - New modules, new features, significant content additions
- `vX.0.0` (Major) - Complete restructuring, breaking changes

**Quick Reference Commands:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlvellotti/claude-code-pm-course](https://github.com/carlvellotti/claude-code-pm-course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
