---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GitHub Feed is a Go CLI tool for monitoring GitHub pull requests and issues across repositories. It tracks contributions, reviews, and assignments with colorized output and real-time progress visualization.

The tool is also called "GitAI" in the README (branding name), but the binary is `github-feed`.

## Build & Run

```bash
# Build the binary
go build -o github-feed .

# Run directly (fetches from GitHub API)
./github-feed

# Run with flags
./github-feed --time 3h        # Show items from last 3 hours
./github-feed --time 2d        # Show items from last 2 days
./github-feed --time 3w        # Show items from last 3 weeks
./github-feed --time 6m        # Show items from last 6 months (default: 1m)
./github-feed --time 1y        # Show items from last year
./github-feed --debug          # Show detailed API logging instead of progress bar
./github-feed --local          # Use local database instead of GitHub API (offline mode)
./github-feed --links          # Show hyperlinks underneath each PR/issue
./github-feed --ll             # Shortcut for --local --links (offline mode with links)
./github-feed --clean          # Delete and recreate the database cache
./github-feed --allowed-repos="owner/repo1,owner/repo2"  # Filter to specific repos
```

## Configuration

The tool requires a GitHub Personal Access Token with `repo` and `read:org` scopes (not needed in `--local` mode). Configuration is loaded from:
1. Environment variables: `GITHUB_TOKEN` or `GITHUB_ACTIVITY_TOKEN`, and `GITHUB_USERNAME` or `GITHUB_USER`
2. Config file at `~/.github-feed/.env` (automatically created on first run)

Database location: `~/.github-feed/github.db` (automatically created on first run)

**First Run**: The tool automatically creates `~/.github-feed/` directory with:
- `.env` file with template for credentials (permissions: 0600)
- `github.db` database for caching GitHub data (permissions: 0666)
- Directory permissions: 0755

## Architecture & Key Components

### Data Flow

#### Online Mode (Default)
1. **Parallel API Fetching**: Six concurrent GitHub search queries for PRs (authored, mentioned, assigned, commented, reviewed, review-requested) - note: no "involved" query exists
2. **Issue Collection**: Four parallel searches for issues (authored, mentioned, assigned, commented)
3. **Database Caching** (db.go): All fetched PRs, issues, and comments are automatically saved to `~/.github-feed/github.db` BBolt database
4. **Cross-Reference Detection**: Links issues to PRs by checking PR/issue bodies and comments for references
5. **Display Rendering**: Separates items into sections by state (open/closed for PRs with merged as subset of closed) with colorized output
6. **Progress Tracking**: Dynamic progress bar that adjusts total count as pagination and additional API calls are discovered
7. **Error Handling**: Infinite retry with exponential backoff for all API calls, handling rate limits gracefully

#### Offline Mode (`--local`)
1. **Database Loading**: Reads all PRs and issues from `~/.github-feed/github.db` instead of making API calls
2. **Data Conversion**: Converts database records to PRActivity and IssueActivity structures
3. **Display Rendering**: Same rendering logic as online mode, showing all cached data
4. **No API Calls**: Completely offline, no GitHub token required

### Core Data Structures

**Config**: Global configuration structure (main.go:46-57):
- Consolidates all application settings (debug mode, local mode, time range, etc.)
- Shared across the application via global `config` variable
- Includes client, database, progress, and context references
- Fields: debugMode, localMode, showLinks, timeRange, username, allowedRepos (map[string]bool), client, db, progress, ctx

**PRActivity**: Represents a PR with metadata (main.go:22-30):
- Label: How the user is involved (e.g., "Authored", "Reviewed")
- Owner/Repo: Repository identification
- PR: GitHub PullRequest object (pointer to github.PullRequest)
- UpdatedAt: Last update timestamp (time.Time)
- HasUpdates: True if API version is newer than cached version
- Issues: Slice of linked IssueActivity that reference this PR

**IssueActivity**: Represents an issue with similar metadata structure (main.go:32-39):
- Label: How the user is involved
- Owner/Repo: Repository identification
- Issue: GitHub Issue object (pointer to github.Issue)
- UpdatedAt: Last update timestamp
- HasUpdates: True if API version is newer than cached version

**Progress**: Thread-safe progress tracking with colored bar display (main.go:41-44):
- Uses `atomic.Int32` for both `current` and `total` fields (no mutexes needed)
- Dynamically adjusts total as pagination and additional API calls are discovered
- Updates in real-time across all goroutines
- Provides visual feedback with color-coded completion status (red <33%, yellow <66%, green >=66%)
- Supports warning messages during retries via `displayWithWarning()` method (main.go:149-159)
- Methods: increment(), addToTotal(n int), buildBar(), display(), displayWithWarning(message string)

**Database**: BBolt wrapper providing structured storage (db.go:20-22):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zveinn/git-feed](https://github.com/zveinn/git-feed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
