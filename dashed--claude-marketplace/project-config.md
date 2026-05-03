---
trigger: always_on
description: This document contains conventions and guidelines for developing the claude-marketplace project.
---

# Development Guide

This document contains conventions and guidelines for developing the claude-marketplace project.

## Versioning and Release Process

### Semantic Versioning

This project follows [Semantic Versioning 2.0.0](https://semver.org/):

**Format**: `MAJOR.MINOR.PATCH` (e.g., `0.2.0`)

**When to bump each version**:
- **MAJOR** (X.0.0): Breaking changes that are not backward compatible
  - Changes to marketplace.json structure that break existing plugins
  - Removal of supported features
  - API changes that require plugin updates

- **MINOR** (0.X.0): New features that are backward compatible
  - Adding new validation systems
  - Adding new plugins to the marketplace
  - Adding new Makefile targets or tooling
  - New documentation or schemas

- **PATCH** (0.0.X): Bug fixes and minor improvements
  - Fixing validation bugs
  - Documentation corrections
  - Dependency updates (non-breaking)
  - Performance improvements

### Version Bump Checklist

When releasing a new version, follow these steps:

1. **Update CHANGELOG.md**
   - Document all changes under `## [Unreleased]` section
   - Categorize changes: Added, Changed, Deprecated, Removed, Fixed, Security
   - Move `## [Unreleased]` content to new version section with date
   - Format: `## [X.Y.Z] - YYYY-MM-DD`
   - Update version comparison links at bottom

2. **Update individual skill changelogs (if applicable)**
   - Update `./changelogs/<skill-name>.md` for any skills that were added, updated, or modified
   - Document skill-specific changes from marketplace perspective
   - Use skill version from marketplace.json
   - See "Individual Skill Changelogs" section below for details

3. **Update .claude-plugin/marketplace.json**
   - Update `metadata.version` field to new version number
   - Example: `"version": "0.2.0"`

4. **Update README.md**
   - Update version in the "Version" section to match marketplace version
   - Add new skills to "Available Skills" table if any were added
   - Ensure skill descriptions and links are accurate

5. **Run validation**
   ```bash
   make validate
   ```
   Ensure all checks pass before proceeding.

6. **Create git commit**
   ```bash
   git add CHANGELOG.md changelogs/ .claude-plugin/marketplace.json README.md
   git commit -m "chore: bump version to vX.Y.Z"
   ```
   Note: Add README.md to git add if it was updated in step 4

7. **Create git tag**
   ```bash
   git tag -a vX.Y.Z -m "Release vX.Y.Z"
   ```
   Note: Use `v` prefix for tags (e.g., `v0.2.0`)

8. **Push changes**
   ```bash
   git push origin master
   git push origin vX.Y.Z
   ```

### Changelog Conventions

Follow [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format:

**Categories** (in order):
- **Added**: New features
- **Changed**: Changes to existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Removed features
- **Fixed**: Bug fixes
- **Security**: Security vulnerability fixes

**Writing good changelog entries**:
- Use present tense ("Add feature" not "Added feature")
- Be specific and concise
- Focus on user-facing changes
- Include tool/file names when relevant
- Group related changes together

**Example**:
```markdown
### Added
- Static validation system with comprehensive checks for marketplace integrity
- JSON schemas for validation: plugin-schema.json, marketplace-schema.json
- Makefile with targets for validation, testing, and linting

### Changed
- Modernized validation workflow to use `uv run` pattern
- Removed unnecessary shebang lines from validator scripts
```

### Individual Skill Changelogs

Each skill in the marketplace has its own changelog located in `./changelogs/`.

**Location**: `./changelogs/<skill-name>.md`
- File names match plugin names from marketplace.json (kebab-case)
- Examples: `skill-creator.md`, `git-absorb.md`, `tmux.md`

**Purpose**: Track marketplace-specific changes to individual skills
- When skills are added to the marketplace
- Version updates in marketplace.json
- Marketplace-specific modifications to skill configuration
- Metadata updates (author, license, keywords, etc.)
- **NOT** upstream plugin development (that stays with the plugin)

**Format**: Same as main CHANGELOG.md
- Follow [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format
- Version numbers match the skill's version in marketplace.json
- Dates when changes were made in the marketplace
- Categories: Added, Changed, Deprecated, Removed, Fixed, Security

**When to Update**:
- Adding a new skill to marketplace → Create new changelog file with initial version
- Updating skill version in marketplace.json → Add new version section
- Making marketplace-specific changes → Document in appropriate version section
- Removing a skill → Add "Removed" entry in main CHANGELOG.md

**Relationship to Main CHANGELOG.md**:
- `CHANGELOG.md`: Marketplace-level changes (validation system, tooling, infrastructure, patterns)
- `changelogs/*.md`: Individual skill changes from marketplace perspective
- **Both**: When skill changes establish/demonstrate marketplace patterns or organizational improvements

**When to Update Both**:

Update ONLY `changelogs/<skill-name>.md`:
- Bug fixes in skill documentation
- Minor wording improvements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dashed/claude-marketplace](https://github.com/dashed/claude-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
