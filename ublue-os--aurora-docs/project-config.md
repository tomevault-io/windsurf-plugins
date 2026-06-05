---
trigger: always_on
description: Aurora Documentation is a Docusaurus TypeScript-based documentation site for Aurora Linux (Universal Blue's KDE productivity distribution). The site serves documentation for installation, usage, development, and project information.
---

# Aurora Documentation Site

Aurora Documentation is a Docusaurus TypeScript-based documentation site for Aurora Linux (Universal Blue's KDE productivity distribution). The site serves documentation for installation, usage, development, and project information.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap, build, and test the repository:

- Node.js 18+ is required (v20.19.5+ recommended). Check with `node --version`
- `npm install` -- takes ~52 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
- `npm run build` -- takes ~25 seconds. Production build with Docusaurus.
- `npm run start` -- starts development server on http://localhost:3000 (~15 seconds to start)
- `npm run serve` -- serves production build on http://localhost:3000 (instant startup)

### Alternative workflows:

- **Docker**: `docker compose up` -- downloads Node.js image (~2 minutes first time), installs dependencies (~9 seconds) and starts dev server (~8 seconds). NEVER CANCEL during image download.
- **Bun (CI workflow)**:
  - Install: `curl -fsSL https://bun.sh/install | bash && export PATH="$HOME/.bun/bin:$PATH"`
  - `bun install --frozen-lockfile` -- takes ~34 seconds (without --production flag)
  - `bun run build` -- takes ~26 seconds, same as npm

### Validation commands (ALWAYS run before committing):

- `npm run typecheck` -- TypeScript validation (~2 seconds)
- `npm run prettier` -- auto-format all files (~1.5 seconds)
- `npm run prettier-lint` -- check formatting without fixing (~1.5 seconds)
- Typos checking: Install `cargo install typos-cli` (~84 seconds), then `/home/runner/.cargo/bin/typos --config ./typos.toml ./README.md ./docs` (~0.03 seconds)

### NEVER CANCEL warnings:

- NEVER CANCEL `npm install` - takes up to 60 seconds
- NEVER CANCEL `docker compose up` during image download - takes up to 3 minutes on first run
- NEVER CANCEL `cargo install typos-cli` - takes up to 100 seconds

## Validation

### Manual testing scenarios:

- ALWAYS test the site functionality after making changes by running `npm run serve` and accessing http://localhost:3000 (or alternative port if 3000 is busy)
- Navigate between documentation sections using the sidebar
- Test search functionality (Ctrl+K or click search button)
- Verify all internal links work correctly
- Test responsive design and dark/light mode toggle (click the theme toggle button)
- Verify that "Edit this page" links point to correct GitHub files
- Test code copy buttons in code blocks
- Navigate through at least 2-3 different documentation pages to ensure routing works

### End-to-end validation:

- The site must load and display the Aurora homepage at "/"
- Navigation between sections must work (Installation, User Guides, Developer Experience, etc.)
- All documentation pages must render properly with correct formatting
- Search functionality must be accessible and responsive (opens overlay when clicked)
- Dark and light themes must work correctly (toggle button changes themes)
- External links in footer and header must be accessible
- "Edit this page" links must point to correct GitHub repository files

### Always run before committing:

- `npm run prettier` (auto-fixes formatting)
- `npm run typecheck` (validates TypeScript)
- `npm run build` (ensures production build succeeds)
- Test the site by running `npm run serve` and manual verification

## Common Tasks

The following are outputs from frequently run commands. Reference them instead of running bash commands to save time.

### Repository structure

```
.
├── .devcontainer/          # VS Code dev container config
├── .github/                # GitHub Actions workflows
├── docs/                   # Documentation content (Markdown)
│   ├── dx/                 # Developer experience docs
│   ├── guides/             # User guides
│   ├── lts/                # LTS information
│   ├── project-docs/       # Project information
│   └── reference/          # Reference documentation
├── src/                    # Docusaurus theme customizations
├── static/                 # Static assets (images, etc.)
├── docusaurus.config.ts    # Main Docusaurus configuration
├── sidebars.ts             # Sidebar navigation configuration
├── package.json            # npm dependencies and scripts
└── tsconfig.json           # TypeScript configuration
```

### Key npm scripts from package.json:

```json
{
  "start": "docusaurus start", // Dev server
  "build": "docusaurus build", // Production build
  "serve": "docusaurus serve", // Serve production build
  "typecheck": "tsc", // TypeScript validation
  "prettier": "prettier --write .", // Auto-format
  "prettier-lint": "prettier --check ." // Check formatting
}
```

### Known issues:

- Build warns about broken anchors in `/guides/rescue-mode` and HTML issues in `/guides/layerapp` - these are non-critical
- AVIF image type not supported by image-size library (warns about `/static/img/dino/dino.avif`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ublue-os/aurora-docs](https://github.com/ublue-os/aurora-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
