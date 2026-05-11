---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Project Overview

This is an MCP (Model Context Protocol) server that provides AI agents like Claude with access to PinMeTo's location management platform. It exposes tools for fetching location data, insights from Google/Facebook/Apple, ratings, and keywords.

## Environment Configuration

The server requires these environment variables (loaded via `--env-file=.env.local` in npm scripts):
- `PINMETO_ACCOUNT_ID` - PinMeTo account identifier
- `PINMETO_APP_ID` - OAuth application ID
- `PINMETO_APP_SECRET` - OAuth application secret
- `PINMETO_API_URL` - (Optional, dev only) Override API base URL
- `PINMETO_LOCATION_API_URL` - (Optional, dev only) Override locations API base URL

## Development Commands

### Build and Test
```bash
npm run build              # Clean build with TypeScript compilation
npm run build:test         # Build with timestamped test version
npm test                   # Run vitest tests
npm run pack:test          # Build and pack for testing
```

### Development
```bash
npm run start              # Run built server in development mode
npm run inspector          # Launch MCP inspector for debugging
npm run format             # Format code with Prettier
```

### Release

This project uses [Changesets](https://github.com/changesets/changesets) for version management and changelog generation.

**Adding Changes (Contributors)**:
```bash
npx changeset add          # Add a changeset for your changes
# Select: major/minor/patch
# Write: summary (appears in CHANGELOG)
git add .changeset/ && git commit -m "docs: add changeset"
```

**⚠️ REQUIRED**: Every PR must include a changeset. CI will reject PRs without one.

**Version Guidelines**:
- **patch**: Bug fixes, documentation, internal changes
- **minor**: New features, enhancements (backwards compatible)
- **major**: Breaking changes (API changes, removed features)

**Release Commands**:
```bash
npm run release:prepare    # Preview pending changesets
npm run release:version    # Bump version + update CHANGELOG + README badges
npm run release:draft      # Test, build, pack, create draft GitHub release
npm run release:publish    # Publish the draft release (or use GitHub UI)
npm run clean              # Remove build directory
```

**Release Flow (Maintainers)**:
1. Run `npm run release:prepare` to see pending changes
2. Run `npm run release:version` to apply version bump
3. Commit: `git add -A && git commit -m "chore: release vX.Y.Z"`
4. Run `npm run release:draft` to create draft release
5. Review draft on GitHub, then run `npm run release:publish`
6. Push: `git push && git push --tags`

## Beads Workflow (CRITICAL)

**Bead Statuses:**
| Status | Description |
|--------|-------------|
| `open` | New/pending work, not yet started |
| `in_progress` | Currently being worked on |
| `blocked` | Waiting on dependency or external blocker |
| `closed` | Completed |

**Hierarchical IDs** (for epics with tasks/sub-tasks):
```
bd-a3f8       # Epic
bd-a3f8.1     # Task under epic
bd-a3f8.1.1   # Sub-task under task
```

**Protected Branch Workflow:**

This project uses a separate `beads-sync` branch for beads metadata:
- Beads daemon auto-commits to `beads-sync` (not main)
- Main branch stays protected from beads auto-commits
- Beads changes are merged to main periodically

```bash
# Check sync status
bd sync --status

# Merge beads changes to main (when needed)
bd sync --merge --dry-run  # Preview
bd sync --merge            # Execute merge to main
```

**Important**: `bd sync` commits to `beads-sync` branch, NOT main. Code changes and beads changes are separate workflows.

```bash
# Create a main bead (epic or feature) with description (REQUIRED)
bd create "Implement user authentication" -t epic -p 1 \
  -d "Add OAuth2 authentication flow with JWT tokens. Includes login, logout, and session management.

## Completion Checklist
- [ ] Implementation complete
- [ ] Tests added and passing
- [ ] Manual testing performed
- [ ] README.md updated (if applicable)
- [ ] Other documentation updated (if applicable)"

# Create sub-tasks under the main bead using --parent
bd create "Set up OAuth2 provider" -t task --parent bd-xxxx \
  -d "Configure OAuth2 provider settings and environment variables."
bd create "Implement login endpoint" -t task --parent bd-xxxx \
  -d "Create POST /auth/login endpoint with credential validation."
bd create "Add session management" -t task --parent bd-xxxx \
  -d "Implement JWT token generation and refresh logic."

# Update issues
bd update bd-a1b2 --status in_progress

# Close issues
bd close bd-a1b2 "Completed authentication"
```

**Critical Rules:**
- Create a bead BEFORE starting any work (not after)
- Create a feature branch BEFORE making any code changes
- Do not set a bead to closed before its PR has been approved
- When starting on a new bead, set it to in-progress and assign it to the current git user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PinMeTo/pinmeto-location-mcp](https://github.com/PinMeTo/pinmeto-location-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
