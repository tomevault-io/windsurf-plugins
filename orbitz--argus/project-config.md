---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Argus is a fast, server-rendered GitHub PR review interface. It prioritizes speed and code understanding over interaction density. Built with Fastify, TypeScript, SQLite, and the GitHub API.

## Development Commands

```bash
# Development (watch mode with hot reload)
npm run dev

# Build TypeScript to JavaScript
npm run build

# Production
npm start

# Database migrations
npm run migrate

# Testing
npm test              # Run all tests once
npm run test:watch    # Watch mode
```

## Architecture

### Request Flow

```
HTTP Request → Authentication Middleware → Route Handler
  ↓
GitHub API (via Octokit) + Local Caching (SQLite)
  ↓
Data Processing (markdown, diff parsing, emoji conversion)
  ↓
EJS Server-Side Rendering
  ↓
HTML Response (with optional client-side JS enhancement)
```

### Core Components

**`src/routes/pr.ts`** - The main PR review interface (~600 lines)
- Handles GET `/pr/:owner/:repo/:number` for PR display
- POST endpoints for comments, reviews, inline comments, and merging
- Orchestrates parallel GitHub API calls for PR data
- Tracks PR revisions to detect force pushes
- Caches rendered diffs and PR snapshots by head SHA

**`src/lib/github.ts`** - GitHub API integration layer (~550 lines)
- Wraps Octokit REST API with caching
- ETag-based cache with configurable TTL
- Functions for fetching PRs, files, checks, comments, reviews, commits
- Functions for posting comments, submitting reviews, merging PRs
- All mutations go through GitHub API (Argus is read-mostly)

**`src/lib/diff-parser.ts`** - Unified diff format parser
- Parses git diff patches into structured data (files, hunks, lines)
- Handles binary files, renames, deletions
- Truncates large files based on `maxLinesPerFile` config

**`src/lib/diff-renderer.ts`** - Diff to HTML converter
- Renders parsed diffs as HTML tables with line numbers
- Supports inline comment forms
- Handles comment threads on specific lines
- Generates file sidebar navigation

**`src/lib/syntax-highlighter.ts`** + **`highlight-pool.ts`** / **`highlight-worker.ts`** / **`highlight-core.ts`** - Shiki highlighting
- Tokenizes each side of a file as one document so constructs closed in an elided region still
  highlight correctly, truncated at the last line the diff actually reads
- Shiki's tokenizer is synchronous, so passes of 40+ lines go to a pool of worker threads
  (`HIGHLIGHT_WORKERS`) rather than blocking the event loop for the whole render
- The pool is an optimization, never a dependency: any spawn failure, worker crash, or
  unsupported language falls back to highlighting in-process

**`src/lib/git.ts`** - Git operations via shell commands
- Clones bare repos to `/tmp/argus-git-cache`
- Computes merge-base for PR base tracking
- Generates range-diffs for force push comparison
- Token sanitization in error messages

**`src/lib/markdown.ts`** - Markdown rendering
- Uses `marked` for GitHub-flavored markdown
- Emoji shortcode conversion via `gemoji` (`:thumbsup:` → 👍)
- Custom link renderer (opens in new tab)
- Task list checkbox support

### Database Schema

SQLite with WAL mode. Key tables:

- **api_cache** - GitHub API response caching with ETags
- **pr_snapshots** - Immutable PR data snapshots by head SHA
- **diff_cache** - Rendered diff HTML cache per file
- **pr_revisions** - Timeline of PR head SHAs (force push detection)
- **user_preferences** - User settings (e.g., skip range-diff confirmation)
- **file_reviews** - Per-user "reviewed" state per file, keyed on blob SHA so it survives
  revisions that don't touch the file
- **commit_reviews** - Per-user "reviewed" state per commit. No invalidation needed: a commit
  SHA is its content, so rewritten commits simply start unreviewed

Migrations in `migrations/` directory, applied on startup.

### Authentication

Personal Access Token (PAT) authentication via `GITHUB_TOKEN` environment variable. Token cached as single user object on startup. No OAuth flow currently implemented.

Required GitHub permissions:
- Pull requests: Read/Write
- Contents: Read
- Commit statuses: Read

### Configuration

Environment variables (see `src/config.ts`):

```bash
# Required
GITHUB_TOKEN=github_pat_...

# Optional
PORT=3000
HOST=0.0.0.0
DATABASE_PATH=./data/argus.db
CACHE_TTL=60000          # API cache TTL in ms
HIGHLIGHT_WORKERS=-1     # Syntax-highlighting worker threads. -1 auto-sizes (max 4),
                         # 0 highlights in-process. Each worker costs ~57MB resident.
BASE_URL=http://localhost:3000
```

### Client-Side JavaScript

**`public/js/pr.js`** - Progressive enhancement for PR view
- Polling for PR updates (shows banner when head SHA changes)
- Keyboard shortcuts over "review items" — commits and files are treated alike, in document
  order (commits first): `g` go-to modal, `n`/`p` next/previous unreviewed, `r` toggle
  reviewed, `c` check for updates. The selected item carries `.review-item-current`
- Inline comment form toggling
- Expand/collapse all comments
- Reply button handlers (mention vs quote reply)
- No hydration needed - works without JavaScript

### Templates


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orbitz/argus](https://github.com/orbitz/argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
