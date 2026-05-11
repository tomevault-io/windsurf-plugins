---
trigger: always_on
description: A monorepo for Stash plugins including:
---

# Stash Plugins Monorepo

A monorepo for Stash plugins including:
- **Stash Downloader** - Download images and videos with automatic metadata extraction
- **Stash Browser** - Browse and preview content from external sources

## Documentation

@.claude/project.md
@.claude/architecture.md
@.claude/conventions.md

## Task Tracking

- **See [`TODO.md`](TODO.md)** — All tasks, backlog items, feature ideas, and completed features are tracked here.
- When making changes or planning features, always refer to the TODO.md file for current project priorities and progress.
- Claude: Use information from TODO.md to inform, update, or cross-reference project documentation, code changes, and pull request review summaries where relevant.

## Quick Reference

### Workspace Commands (from root)
| Command                    | Description                       |
|----------------------------|-----------------------------------|
| `npm run build`            | Build all plugins                 |
| `npm run build:downloader` | Build Stash Downloader only       |
| `npm run build:browser`    | Build Stash Browser only          |
| `npm run test`             | Run tests for all plugins         |
| `npm run type-check`       | TypeScript check all plugins      |
| `npm run lint`             | ESLint all plugins                |

### Plugin-Specific Commands (from plugins/stash-downloader/)
| Command                | Description                  |
|------------------------|------------------------------|
| `npm run dev`          | Build with watch mode        |
| `npm run build`        | Production build             |
| `npm run build:stash`  | Build and create plugin folder|
| `npm test`             | Run tests                    |
| `npm run type-check`   | TypeScript check             |
| `npm run lint`         | ESLint check                 |

## Branch & Release Workflow

### Branch Strategy
| Branch  | Purpose                                   |
|---------|-------------------------------------------|
| **dev** | Active development - all work happens here |
| **main**| Stable releases only - merged from dev     |

**ALWAYS develop on `dev` branch.** Never commit directly to `main`.

### Development Flow
```
1. Work on dev branch
2. Push to dev → triggers dev build (Downloader-Dev)
3. When ready for release: merge dev → main, bump version, tag
4. Tag on main → triggers stable release (push to main alone does NOTHING)
5. WAIT for workflow to complete before syncing dev
```

### Component Versions (Independent)
| Component            | Version Location                        | Release Trigger              |
|----------------------|-----------------------------------------|------------------------------|
| **Stash Downloader** | `plugins/stash-downloader/package.json` | `downloader-vX.Y.Z` tag      |
| **Stash Browser**    | `plugins/stash-browser/package.json`    | `browser-vX.Y.Z` tag         |
| **Firefox Extension**| `browser-extension/package.json`        | `extension-vX.Y.Z` tag + AMO |

### Stable vs Dev Builds
| Plugin           | Stable Tag          | Dev Trigger    | Plugin IDs                          |
|------------------|---------------------|----------------|-------------------------------------|
| **Downloader**   | `downloader-vX.Y.Z` | Push to `dev`  | `stash-downloader` / `stash-downloader-dev` |
| **Browser**      | `browser-vX.Y.Z`    | Push to `dev`  | `stash-browser` / `stash-browser-dev` |

Both can be installed simultaneously in Stash (different plugin IDs via YAML filename).  
Both are served from the same `index.yml` - each deploy preserves the other's entry.

### Release Process
```bash
# Release Stash Downloader:
git checkout main && git merge dev
cd plugins/stash-downloader && npm version patch  # or minor/major
git add . && git commit -m "🔖 chore: release downloader-vX.Y.Z"
git tag downloader-vX.Y.Z && git push origin main --tags

# Release Stash Browser:
git checkout main && git merge dev
cd plugins/stash-browser && npm version patch
git add . && git commit -m "🔖 chore: release browser-vX.Y.Z"
git tag browser-vX.Y.Z && git push origin main --tags

# Release Firefox Extension:
git checkout main && git merge dev
cd browser-extension && npm version patch  # auto-syncs manifest.json
git add . && git commit -m "🔖 chore: release extension-vX.Y.Z"
git tag extension-vX.Y.Z && git push origin main --tags
# Then: Download ZIP from GitHub Release → upload to AMO
```

Or use `/release` skill for guided release.

### After Releasing
- **Plugins**: GitHub Actions builds + deploys to GitHub Pages + creates GitHub Release
- **Extension**: GitHub Actions creates Release with ZIP → download and upload to AMO manually
- **Dev sync**: Sync dev with main AFTER workflow completes (to avoid cancelling stable deploy)

**⚠️ WAIT for stable workflow to complete before pushing to dev** - concurrent deploys cancel each other!

## Tech Stack

- **Frontend**: React 18 + TypeScript
- **Build**: Vite (IIFE bundle for Stash plugins)
- **Styling**: Bootstrap utilities (provided by Stash)
- **API**: GraphQL via Apollo Client (provided by Stash)
- **Backend**: Python + yt-dlp for server-side downloads
- **Scraping**: Server-side via yt-dlp Python backend

## Key Constraints

- External deps (React, Apollo) provided by Stash - NOT bundled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Codename-11/Stash-Downloader](https://github.com/Codename-11/Stash-Downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
