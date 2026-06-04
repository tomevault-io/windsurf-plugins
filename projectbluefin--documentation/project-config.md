---
trigger: always_on
description: Generates a report MDX in `reports/`, commits to a branch, creates a PR with auto-merge.
---

# Bluefin Documentation — Agent Instructions

**Repository:** `projectbluefin/documentation`  
**Deployed at:** <https://docs.projectbluefin.io/>  
**Local setup:** Clone locally with `gh repo clone projectbluefin/documentation`  
**Framework:** Docusaurus 3.10.x (TypeScript), React 19, Node 24

---

## Quick Start

```bash
# Install dependencies
npm install --legacy-peer-deps

# Start dev server (fetches data automatically, hot-reload)
npm run start

# Full production build
npm run build

# Type check
npm run typecheck

# Lint
npm run lint
```

Dev server: <http://localhost:3000/>

---

## Git Workflow

**Never push directly to main.** Always work on a topic branch:

```bash
git checkout -b <type>/<short-description>
git add <files>
git commit -m "type(scope): description"
git push -u origin <type>/<short-description>
# Then open a PR — do not merge
```

Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `ci`, `chore`

AI agent attribution (required in every commit footer — exactly ONE trailer, no Co-authored-by):

```
Assisted-by: Claude Sonnet 4.6 via GitHub Copilot
```

**Fork sync:** This is a fork of `projectbluefin/documentation`. Keep in sync:

```bash
git fetch upstream
git checkout main
git reset --hard upstream/main
git push origin main --force-with-lease
```

---

## Validation Gates (all required before committing)

| Check | Command | Blocking? |
|---|---|---|
| TypeScript | `npm run typecheck` | YES |
| ESLint | `npm run lint` | YES |
| Prettier | `npm run prettier-lint` | No (warnings only in CI) |
| Build | `npm run build` | YES |

CI enforces TypeScript and ESLint as hard failures. Prettier is warnings-only.

---

## Repository Structure

```
docs/                   # User documentation (Markdown/MDX)
blog/                   # Blog posts with frontmatter + authors.yaml
reports/                # Monthly auto-generated report MDX posts
src/
  components/           # React components (see Components section)
  config/               # packageConfig.ts — centralized package tracking
  pages/                # Custom Docusaurus pages
    changelogs.tsx       # /changelogs page — only custom page, all others are docs/ MDX
  types/                # TypeScript type definitions
    sbom.ts              # SBOM attestation types
    sbom-attestations.d.ts  # Ambient module declaration (allows missing file at tsc time)
    data.d.ts            # General data types
    theme.d.ts           # Docusaurus theme type augmentations
  css/                  # custom.css
  theme/                # Swizzled Docusaurus components
    DocItem/Footer/      # Adds PageContributors to every doc page
scripts/                # Data-fetch and utility scripts (see Data Pipeline section)
  lib/                  # Shared library modules for generate-report
static/
  data/                 # Auto-generated JSON (gitignored — except sbom-attestations.json and sbom-attestations-frontend.json)
  feeds/                # Auto-generated release feeds (gitignored)
  img/                  # Static images
.github/
  workflows/            # CI/CD workflows (see CI/CD section)
  agents/               # GitHub Copilot agent definitions
  prompts/              # Copilot prompt files
docusaurus.config.ts    # Main Docusaurus configuration
sidebars.ts             # Sidebar navigation
Justfile                # build, serve recipes
```

---

## Data Pipeline

### How data reaches the site

The site fetches all data at **build time** via npm scripts. No runtime API calls from the browser (except `GnomeExtensions.tsx` which fetches live from extensions.gnome.org, and the `ProjectCard.tsx` runtime fallback for missing repo stats).

#### Standard data pipeline (`npm run fetch-data`)

Runs automatically during `npm run start` and `npm run build`.

| Script | Output | What it fetches |
|---|---|---|
| `fetch-feeds.js` | `static/feeds/bluefin-releases.json`, `static/feeds/bluefin-lts-releases.json` | GitHub Atom release feeds from ublue-os/bluefin and ublue-os/bluefin-lts |
| `fetch-playlist-metadata.js` | `static/data/playlist-metadata.json` | YouTube playlist thumbnails/descriptions |
| `fetch-github-profiles.js` | `static/data/github-profiles.json` | GitHub user profiles (~80 users for donations page) |
| `fetch-github-repos.js` | `static/data/github-repos.json` | GitHub repo stars/forks for projects page |
| `fetch-contributors.js` | `static/data/file-contributors.json` | Per-file git contributors (used by DocItem/Footer) |
| `fetch-gnome-extensions.js` | `static/data/gnome-extensions.json`, `static/img/extensions/` | GNOME extension metadata + screenshots for `/tips/` page |
| `fetch-pin-state.js` | `static/data/stream-pins.json` | Current pinned stream versions |
| `fetch-github-driver-versions.js` | `static/data/driver-versions.json` | Kernel/Mesa/NVIDIA/GNOME version history per stream from GitHub releases |
| `fetch-github-images.js` | `static/data/images.json` | OCI image catalog — streams, versions, bootc switch commands, download counts. Reads `sbom-attestations.json` and overlays package versions from SBOM data |
| `fetch-firehose.js` | `static/data/firehose-apps.json` | Flatpak/app release data from castrojo/bluefin-releases; enriched with SBOM OS release entries |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectbluefin/documentation](https://github.com/projectbluefin/documentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
