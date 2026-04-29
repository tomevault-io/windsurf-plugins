---
trigger: always_on
description: **al-folio** is a simple, clean, and responsive [Jekyll](https://jekyllrb.com/) theme for academics and researchers. It enables users to create professional portfolio and blog websites with minimal configuration. The repository serves both as a template and as a reference implementation.
---

# Copilot Coding Agent Instructions

## Repository Overview

**al-folio** is a simple, clean, and responsive [Jekyll](https://jekyllrb.com/) theme for academics and researchers. It enables users to create professional portfolio and blog websites with minimal configuration. The repository serves both as a template and as a reference implementation.

- **Type:** Jekyll static site generator template
- **Target Users:** Academics, researchers, and professionals
- **Key Features:** CV display, publication bibliography, blog posts, projects, news/announcements, course listings

## Tech Stack & Versions

**Core Technologies:**

- **Jekyll:** v4.x (Ruby static site generator)
- **Ruby:** 3.3.5 (primary CI/CD version), 3.2.2 (some workflows)
- **Python:** 3.13 (for nbconvert, jupyter notebook support)
- **Node.js:** Latest (for purgecss and prettier)
- **Docker:** Uses prebuilt image `amirpourmand/al-folio:v0.16.3` (Ruby slim-based)

**Build Dependencies (from Gemfile):**

- `classifier-reborn` – Related posts calculation
- `jekyll-archives-v2` – Archive page generation
- `jekyll-jupyter-notebook` – Jupyter notebook embedding
- `jekyll-minifier` – CSS/JS minification
- `jekyll-paginate-v2` – Pagination
- `jekyll-scholar` – Bibliography management
- `jekyll-tabs` – Tab UI components
- `jekyll-toc` – Table of contents generation
- `jemoji` – Emoji support
- Multiple other specialized jekyll plugins

**Code Quality Tools:**

- **Prettier:** v3.8.0+ with `@shopify/prettier-plugin-liquid` – Code formatter (mandatory for PRs)
- **Purgecss:** CSS purification for production builds

## Building & Local Development

### Docker (Recommended Approach)

**Always use Docker for local development.** This ensures consistency with CI/CD and avoids Ruby/Python environment issues.

**Initial Setup:**

```bash
docker compose pull                    # Pull prebuilt image
docker compose up                      # Start development server
# Site runs at http://localhost:8080
```

**Rebuilding with Updated Dependencies:**

```bash
docker compose up --build              # Rebuilds Docker image from Dockerfile
docker compose up --force-recreate     # Forces complete rebuild
```

**For slim Docker image (if image size is critical):**

```bash
docker compose -f docker-compose-slim.yml up
```

**If Docker build fails:**

- Check disk space and available RAM
- Kill any existing jekyll processes: `docker compose down`
- For M1/M2 Mac: Ensure Docker Desktop is up-to-date
- Linux users may need Docker group permissions: `sudo usermod -aG docker $USER` (then logout/login)

### Bundle/Jekyll (Legacy, Use Docker Instead)

```bash
bundle install                         # Install Ruby gems
pip install jupyter                    # Install Python dependencies
bundle exec jekyll serve --port 4000   # Run at http://localhost:4000
```

### Important Build Requirements

- **ImageMagick must be installed** – Required for image processing plugins
  - Docker: Installed automatically
  - Local: `sudo apt-get install imagemagick` (Linux) or `brew install imagemagick` (Mac)
- **nbconvert must be upgraded before build** – `pip3 install --upgrade nbconvert`
- **Always set JEKYLL_ENV=production for production builds** – Required for CSS/JS minification

## Project Layout & Key Files

### Root Directory Structure

- `_bibliography/papers.bib` – BibTeX bibliography for publications
- `_config.yml` – **Primary configuration file** (title, author, URLs, baseurl, feature flags)
- `_data/` – YAML data files (socials.yml, coauthors.yml, cv.yml, citations.yml, venues.yml, repositories.yml)
- `_includes/` – Reusable Liquid template components
- `_layouts/` – Page layout templates (about.liquid, post.liquid, bib.liquid, distill.liquid, cv.liquid, etc.)
- `_news/` – News/announcement entries
- `_pages/` – Static pages (about.md, cv.md, publications.md, projects.md, teaching.md, etc.)
- `_posts/` – Blog posts (format: YYYY-MM-DD-title.md)
- `_projects/` – Project showcase entries
- `_sass/` – SCSS stylesheets
- `_scripts/` – JavaScript files for functionality
- `_teachings/` – Course and teaching entries
- `assets/img/` – Images, profile pictures
- `docker-compose.yml` – Docker compose configuration
- `Dockerfile` – Docker image definition
- `Gemfile` & `Gemfile.lock` – Ruby dependency specifications
- `package.json` – Node.js dependencies (prettier only)
- `purgecss.config.js` – PurgeCSS configuration for production CSS optimization

### Configuration Priority

When making changes:

1. **Always start with `_config.yml`** for site-wide settings
2. **Feature flags are in `_config.yml`** – Look for `enabled: true/false` options
3. **Social media links:** `_data/socials.yml`
4. **Content data:** Respective `_data/*.yml` files
5. **Styling:** `_sass/` directory (uses SCSS)

## CI/CD Pipeline & Validation

### GitHub Workflows (in `.github/workflows/`)

- **deploy.yml** – Main deployment workflow (runs on push/PR to main/master)
  - Sets up Ruby 3.3.5, Python 3.13
  - Installs imagemagick, nbconvert

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aitechroberts/aitechroberts.github.io](https://github.com/aitechroberts/aitechroberts.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
