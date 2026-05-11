---
trigger: always_on
description: **gh-manager-cli** is an interactive CLI tool built with Ink (React for terminals) that helps users manage GitHub repositories. This project provides a terminal-based UI for browsing, searching, and managing personal GitHub repos with real-time API integration.
---

# AGENTS.md - gh-manager-cli Project Memory & Instructions

## Project Overview

**gh-manager-cli** is an interactive CLI tool built with Ink (React for terminals) that helps users manage GitHub repositories. This project provides a terminal-based UI for browsing, searching, and managing personal GitHub repos with real-time API integration.

### Current Status: Active Development
- ✅ Core repository listing functionality
- ✅ GitHub GraphQL API integration with Apollo Client caching
- ✅ Interactive terminal UI with Ink
- ✅ OAuth and PAT authentication with secure storage
- ✅ Infinite scroll with smart prefetching
- ✅ Repository management (delete, archive, visibility change)
- ✅ GitHub Enterprise support with Internal visibility
- ✅ Organization switching and context management
- ✅ Fork synchronization with upstream
- ✅ Semantic release automation and CI/CD workflows
- ✅ Automated changelog generation and PR title management
- 🔧 Automated test suite expansion (ongoing)
- 🔧 Cross-terminal rendering optimization

**For current version and recent changes, see [CHANGELOG.md](./CHANGELOG.md)**

## Repository Structure

```
gh-manager-cli/
├── src/
│   ├── index.tsx              # CLI entry, error boundaries, renders App
│   ├── ui/
│   │   ├── App.tsx            # Token bootstrap and routing
│   │   └── RepoList.tsx       # Repository list UI, key handling, infinite scroll
│   ├── github.ts              # Octokit GraphQL client and queries
│   ├── config.ts              # Read/write config and token management
│   └── types.ts               # TypeScript type definitions
├── dist/                      # Built output (gitignored)
├── package.json               # NPM package config with semantic-release
├── tsconfig.json              # TypeScript configuration
├── tsup.config.ts             # Build configuration (shebang-preserved CJS)
├── CHANGELOG.md               # Generated changelog (semantic-release)
├── README.md                  # User documentation
├── LICENSE                    # MIT License
├── AGENTS.md                  # This file - project memory/instructions
├── .gitignore                 # Git ignore patterns
└── .github/
    ├── workflows/
    │   ├── automated-release.yml    # Semantic release on main push
    │   └── pr-title-manager.yml     # PR title automation
    └── scripts/
        └── normalize-pr-title.js    # PR title normalization logic
```

## Core Features

### Main Script (`src/index.tsx`)
- **Language:** TypeScript with React/Ink
- **Dependencies:** 
  - `@octokit/graphql` for GitHub API
  - `ink` (React-based TUI)
  - `chalk` for terminal colors
  - `ink-spinner` for loading states
  - `ink-text-input` for user input
  - `env-paths` for cross-platform config storage
- **Build:** tsup with esbuild

### Key Features
- OAuth and PAT authentication: prompt → validate → persist (0600 perms on POSIX)
- List personal and organization repos with metadata (name, description, stars, forks, etc.)
- Full keyboard navigation with extensive shortcuts
- Smart infinite scroll with 80% prefetch trigger
- Server-side search with Apollo Client caching
- Repository actions: delete, archive/unarchive, change visibility, sync forks
- Organization and Enterprise GitHub support
- Modal-based UI for sorting, filtering, and actions
- Persistent UI preferences (sort, density, visibility filter, fork tracking)
- Real-time rate limit monitoring for GraphQL and REST APIs

### Planned Enhancements
See the living roadmap in [TODOs.md](./TODOs.md) for the canonical, up-to-date list. Key near-term items include:
- Repository renaming
- Bulk selection and actions
- Copy repository URL to clipboard
- Optional OS keychain support (via `keytar`)

## Configuration & Token Storage

- Reads token from `process.env.GITHUB_TOKEN` or `process.env.GH_TOKEN` first.
- Fallback to config file: created on first successful validation.
- Config path via `env-paths('gh-manager-cli').config`:
  - macOS: `~/Library/Preferences/gh-manager-cli/config.json`
  - Linux: `~/.config/gh-manager-cli/config.json`
  - Windows: `%APPDATA%\gh-manager-cli\config.json`
- Permissions:
  - POSIX: `chmod 600` after writing file.
- Shape:
  ```json
  { "token": "<pat>", "tokenVersion": 1 }
  ```
- PAT scopes:
  - For listing all personal repos including private: classic PAT with `repo` scope (read is sufficient).
  - If only public repos are needed, a token with public-repo read may suffice, but `repo` is recommended.

## GitHub API Details

- GraphQL query against `viewer.repositories` with `ownerAffiliations: OWNER` and `orderBy: UPDATED_AT DESC`.
- Page size: 50 per request.
- On each page fetch, also read `totalCount` to reflect newly created repos.
- Selected fields: name/nameWithOwner/description/visibility/isPrivate/isFork/isArchived/stargazerCount/forkCount/primaryLanguage/updatedAt/pushedAt/diskUsage.

## Controls

- Up/Down: move selection
- PageUp/PageDown: jump ±10
- `Ctrl+G`: jump to top
- `G`: jump to bottom
- `/`: search mode (3+ characters for server-side search, Esc cancels)
- `S`: sort modal (updated, pushed, name, stars)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wiiiimm/gh-manager-cli](https://github.com/wiiiimm/gh-manager-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
