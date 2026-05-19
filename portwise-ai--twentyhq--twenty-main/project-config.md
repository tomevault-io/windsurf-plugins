---
trigger: always_on
description: Process and guidelines for creating release changelogs for Twenty CRM
---

# Twenty Release Changelog Process

Complete guide for creating release changelogs, including codebase research, file structure, and content guidelines.

## Prerequisites

Before starting, gather the following information:

### Required Information

**Version Number**: `{VERSION}` (e.g., 1.9.0, 2.0.0, 2.1.0)

**Release Date**: Use today's date in format: YYYY-MM-DD

### Changes/Features to Document

List the features and changes to include in this release:

1. **Feature Name**: ______________________________
   - Brief description: ______________________________
   - Related area (workflow, UI, backend, etc.): ______________________________

2. **Feature Name**: ______________________________
   - Brief description: ______________________________
   - Related area: ______________________________

3. **Feature Name**: ______________________________
   - Brief description: ______________________________
   - Related area: ______________________________

## Codebase Research Guide

If feature descriptions are not provided or need enhancement, research the codebase:

### Where to Look

**For Workflow Features:**
- Frontend: `packages/twenty-front/src/modules/workflow/`
- Backend: `packages/twenty-server/src/modules/workflow/`
- Components: `packages/twenty-front/src/modules/workflow/components/`

**For UI/UX Changes:**
- Components: `packages/twenty-front/src/modules/ui/`
- Layout: `packages/twenty-front/src/modules/layout/`
- Design system: `packages/twenty-ui/src/`

**For Backend/API Features:**
- Server modules: `packages/twenty-server/src/modules/`
- Entities: `packages/twenty-server/src/entities/`
- Services: Look for `*.service.ts` files

**For Database/ORM Changes:**
- Migrations: `packages/twenty-server/src/database/typeorm/`
- Entities: `packages/twenty-server/src/entities/`

### Research Commands

```bash
# Find recent merged PRs (adjust date as needed)
gh pr list --search "merged:>2025-10-01" --limit 50 --state merged

# View recent commits
git log --since="2 weeks ago" --oneline --no-merges

# View commits between releases (replace with actual release tags)
git log v1.7.0..v1.8.0 --oneline

# Search for specific feature keywords in code
grep -r "iterator" packages/twenty-front/src/modules/workflow/
grep -r "bulk select" packages/twenty-front/src/modules/workflow/

# Find recent changes in specific directory
git log --since="2 weeks ago" --oneline -- packages/twenty-front/src/modules/workflow/
```

### Using Codebase Search

Use the AI codebase search to find:
- "How does the workflow iterator node work?"
- "Where is bulk select implemented for workflows?"
- "What changes were made to the search node limit?"

## Step-by-Step Process

### 1. Setup Git Branch

**IMPORTANT**: Always start from an up-to-date main branch to avoid merge conflicts and ensure the changelog is based on the latest code.

```bash
cd /Users/thomascolasdesfrancs/code/twenty
git checkout main
git pull origin main
git checkout -b {VERSION}
```

Replace `{VERSION}` with the actual version number (e.g., `1.9.0`)

⚠️ **Do this first** before making any file changes. This ensures your branch is based on the latest main.

### 2. Create File Structure

**Create changelog file:**
- Path: `packages/twenty-website/src/content/releases/{VERSION}.mdx`
- Example: `packages/twenty-website/src/content/releases/1.9.0.mdx`

**Create image folder:**
- Path: `packages/twenty-website/public/images/releases/{MINOR_VERSION}/`
- Example for version 1.9.0: `packages/twenty-website/public/images/releases/1.9/`
- Example for version 2.0.0: `packages/twenty-website/public/images/releases/2.0/`

```bash
# Create the image folder
mkdir -p packages/twenty-website/public/images/releases/{MINOR_VERSION}
```

### 3. Move Illustration Files

**Source:** `/Users/thomascolasdesfrancs/Downloads/🆕`

**Destination:** `packages/twenty-website/public/images/releases/{MINOR_VERSION}/`

**Naming Convention:** `{VERSION}-descriptive-name.png`

Examples:
- `1.9.0-feature-name.png`
- `1.9.0-another-feature.png`

```bash
# Move and rename files
cp ~/Downloads/🆕/source-file.png packages/twenty-website/public/images/releases/{MINOR_VERSION}/{VERSION}-feature-name.png
```

### 4. Research Features (if needed)

If descriptions are not provided:
1. Use the research commands above to find recent PRs and commits
2. Search the codebase for feature-related code
3. Read PR descriptions for context
4. Check component comments and documentation

### 5. Write Changelog Content

Create the MDX file with this structure:

```markdown
---
release: {VERSION}
Date: {YYYY-MM-DD}
---

# Feature 1 Name

Short description explaining what the feature does and why it's useful. Keep it user-focused and concise (1-2 sentences).

![](/images/releases/{MINOR_VERSION}/{VERSION}-feature-1.png)

# Feature 2 Name

Another short description of the second feature.

![](/images/releases/{MINOR_VERSION}/{VERSION}-feature-2.png)

# Feature 3 Name

Description of the third feature.

![](/images/releases/{MINOR_VERSION}/{VERSION}-feature-3.png)
```

**Style Guidelines:**
- Use H1 (`#`) for feature names
- Keep descriptions to 1-2 sentences
- Focus on user benefits, not technical implementation
- Use active voice
- Start with what the user can now do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portwise-ai/twentyhq__twenty.main](https://github.com/portwise-ai/twentyhq__twenty.main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
