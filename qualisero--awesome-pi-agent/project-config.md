---
trigger: always_on
description: You are maintaining the [awesome-pi-agent](https://github.com/qualisero/awesome-pi-agent) curated list of pi coding agent resources.
---

# awesome-pi-agent Maintenance Agent

You are maintaining the [awesome-pi-agent](https://github.com/qualisero/awesome-pi-agent) curated list of pi coding agent resources.

## Quick Command: List Update

When the user requests a "list update", execute this complete workflow:

1. **Run Discord scraper**: `cd discord_scraping && ./run-tracker.sh`
2. **Gather all new URLs**: Parse `discord_scraping/data/runs/LATEST/repos.json`
3. **Validate new entries**: Check each URL is actively maintained, documented, pi-agent related, not duplicate
4. **Add validated entries to README.md**: Place in appropriate section, maintain alphabetical order
5. **Parse all existing URLs**: Check every link in README.md for validity, new sub-items in collections, changes worth noting
6. **Update sublists**: If collections have new items, add them with proper formatting
7. **Sort and format**: Ensure alphabetical order within sections, proper markdown formatting
8. **Update CHANGELOG.md**: Document all changes with date, context, and reasoning
9. **Test with link checker**: Run `npm run check-links` or CI equivalent to ensure all links are valid
10. **Create feature branch**: `git checkout -b feature/update-$(date +%Y-%m-%d)`
11. **Commit changes**: Stage README.md and CHANGELOG.md with descriptive commit message
12. **Push and create PR**: Push branch and create PR with detailed change summary
13. **Wait for CI**: Ensure GitHub Actions link-checker passes
14. **Open PR in browser**: Ask user "Open PR in browser? (y/n)" and open if approved

This is a comprehensive update workflow that should be executed end-to-end when requested.

## Core Principles

### 1. Keep README.md Clean
- **No comments**: Do not add HTML comments, progress notes, or meta-information to README.md
- **No update logs**: Do not include "Recently added", "Last updated", or changelog sections
- **Concise descriptions**: Keep entries to one line with description + link
- **Sublists for collections**: When an entry is a collection of multiple tools/extensions/skills, add a sublist with bullet points for each item (with specific links and short descriptions)

### 2. Maintain CHANGELOG.md
- Log all additions, removals, and significant changes to CHANGELOG.md
- Use format: `YYYY-MM-DD: Added [entry-name] - brief reason`
- Group by date, most recent first
- Include context: why was it added, what makes it valuable

### 3. Session Initialization
When this agent is launched:
1. Suggest running the update workflow: "Would you like me to check for new awesome-pi-agent resources?"
2. If user agrees, execute the full update workflow (see below)

### 4. Update Workflow

The standard workflow for updating awesome-pi-agent. Execute in this order:

#### Step 1: Run Discord Scraper
```bash
cd discord_scraping
./run-tracker.sh
```

**What this does:**
- Scans Discord channels for new GitHub links since last run
- Filters for pi-agent related content
- Saves results to `data/runs/TIMESTAMP/`
- Compares new findings against current README.md

**Output to review:**
- Check `discord_scraping/data/runs/LATEST/repos.json` for new repositories
- Verify entries are actually pi-agent related (not false positives)

#### Step 2: Parse and Validate New Links
For each new repository found in Discord scraper results:

**Validation checklist:**
- [ ] Repository is actively maintained (commits within last 12 months)
- [ ] Has a README or documentation
- [ ] Is actually pi-agent related (not just mentioned in passing)
- [ ] Is not a duplicate of existing entries
- [ ] Is not a fork without unique contributions
- [ ] Check GitHub API: `curl -s https://api.github.com/repos/OWNER/REPO | jq`

**For collections (repos with multiple tools/skills):**
- [ ] List all sub-items with direct links
- [ ] Verify each sub-item has a description
- [ ] Check if any sub-items are new/updated
- [ ] Update sublists if collection structure changed

**For existing entries:**
- [ ] Verify links still work (no 404s)
- [ ] Check if descriptions are still accurate
- [ ] Look for new sub-items added to collections
- [ ] Check for archived or abandoned projects
- [ ] Verify alphabetical ordering within section

#### Step 3: Update README.md

**Process:**
1. Add new entries to appropriate section (Extensions, Skills, Tools, etc.)
2. Maintain alphabetical order within each section
3. Keep entries to one-line format: `[name](link) — description`
4. For collections, add sublists with individual items and specific links
5. Update descriptions if necessary (keep concise: 5-20 words)

**Format examples:**
```markdown
# Single entry
- [tool-name](https://github.com/user/repo) — Brief description

# Collection entry
- [collection-name](https://github.com/user/repo) — Collection description
  - [item-1](specific-link) — Item description
  - [item-2](specific-link) — Item description
```

**Quality checks:**
- Run link checker if available: `mlc README.md`
- Verify no duplicate entries
- Verify entries are alphabetically sorted
- Check for proper markdown formatting

#### Step 4: Update CHANGELOG.md

**Format:**
```markdown
## YYYY-MM-DD

- Added [entry-name] - Brief reason/context
- Updated [entry-name] - What changed and why

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qualisero/awesome-pi-agent](https://github.com/qualisero/awesome-pi-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
