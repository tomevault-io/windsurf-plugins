---
trigger: always_on
description: > **Note for human readers:** This file contains instructions for Claude Code and other AI assistants when editing this repository. It's not part of the game guide itself - see [README.md](README.md) to start reading the guide.
---

# CLAUDE.md

> **Note for human readers:** This file contains instructions for Claude Code and other AI assistants when editing this repository. It's not part of the game guide itself - see [README.md](README.md) to start reading the guide.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow for AI Assistants

### Initial Session Protocol

**REQUIRED:** At the start of every new session working with this repository, follow this protocol:

1. **Read README.md** - Understand the guide structure, navigation, and current version
2. **Read CLAUDE.md** (this file) - Review editing conventions and repository structure
3. **Read CHANGELOG.md** - Review recent changes to understand what's been updated

**Why this matters:**

- README.md provides the current version number and guide overview
- CLAUDE.md contains critical editing instructions and style guidelines
- CHANGELOG.md shows what's changed recently to avoid redundant work

### Making Changes Workflow

When the user requests changes to the guide, follow this workflow:

#### Step 1: Identify Target File(s)

- **Chapter content**: Edit specific chapter file (e.g., `01-chapter-01.md`, `03-chapter-09.md`)
- **Core mechanics**: Edit `00-introduction.md`
- **Reference tables**: Edit appropriate appendix (`appendix-a-slimes.md`, `appendix-b-items.md`, etc.)
- **Ranch progression**: Edit `appendix-l-plot-overview.md` (treat changes here as main guide content — version + CHANGELOG update required)
- **Navigation/meta**: Edit `README.md`
- **New chapter / new CHANGELOG entry**: start from `.templates/chapter-template.md` or `.templates/changelog-entry-template.md`

#### Step 2: Make Content Edits

- Follow the [Writing Style](#writing-style) guidelines below
- Maintain consistent formatting with existing content
- Use tables for comparative data in appendices
- Include warning callouts for critical information

#### Step 3: Update Version Information

**If content was changed** (not just typo fixes):

1. Update `00-introduction.md` version header:
   - Increment version number (0.1 → 0.2)
   - Update "Last Updated" date to current date
   - Create a descriptive version name (e.g., "Saber Strategies Edition")
2. Do NOT update version in individual chapter files - it only lives in `00-introduction.md`

#### Step 4: Update CHANGELOG.md

**REQUIRED for ALL content changes:**

1. Add a new version section at the top of CHANGELOG.md
2. Document what was changed:
   - New content added
   - Corrections made
   - Updates to existing strategies
   - Files modified
3. Use the existing format as a template
4. Be specific about what changed (not just "updated Chapter 5")

**Example CHANGELOG entry:**

```markdown
## [0.2] - 2025-10-25 - Saber Strategies Edition

### Changed
- **Chapter 7** (02-chapter-07.md): Updated Saber Slime acquisition strategy
  - Added alternate route through northern passage
  - Corrected Thundercluck spawn location
- **Appendix A** (appendix-a-slimes.md): Fixed Saber Slime favorite food

### Fixed
- Corrected markdown link in Chapter 7 to appendix-a-slimes.md
```

#### Step 5: Update the mirror files (if version changed)

If you updated the version in Step 3, also update the header in each of the sanctioned mirror files so they stay in sync with `00-introduction.md`:

- `README.md` — version header at the top (only spot; bottom Version Information section is version-free)
- `steam/SECTION-01-INTRO.txt` — version header at the top (BBCode)
- `steam/SECTION-18-APPENDICES-2.txt` — version footer line at the bottom (BBCode; this is Part 2 of the split appendices section)

The `version-sync.yml` CI job will fail if any file outside this list (plus `CHANGELOG.md`) hardcodes a `Version 0.X` string.

### Quality Checks

Before considering changes complete, verify:

1. **Link Integrity**: Do all markdown links still work?
   - Chapter-to-chapter references
   - Chapter-to-appendix references
   - README navigation links

2. **Cross-Reference Validation**:
   - If you changed appendix data (slime stats, prices, locations), check if chapters reference that data
   - If you changed chapter content, verify appendix-l-plot-overview.md still matches
   - Ensure consistency between related sections

3. **Formatting Consistency**:
   - Tables are properly formatted with aligned columns
   - Bold/italic formatting matches style guide
   - Heading hierarchy is correct (##, ###, ####)
   - Code blocks and callouts are properly formatted

4. **Data Accuracy**:
   - Numbers and statistics are consistent across files
   - Largo combinations make sense (correct slime pairings)
   - Revenue calculations match the plotted corrals

### Git and Version Control

**CRITICAL RULES - READ CAREFULLY:**

- **NEVER commit or push changes automatically** - ONLY commit/push when the user EXPLICITLY asks you to
- **DO NOT commit after making edits** - Just make the edits and STOP
- **DO NOT push after committing** - Only push when explicitly requested
- The user will tell you when they want changes committed and pushed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skelhammer/slime-rancher-2-guide](https://github.com/skelhammer/slime-rancher-2-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
