---
trigger: always_on
description: Generates interactive map of talk locations:
---

# AGENTS.md - Academic Pages Jekyll Site

## Project Overview

This is an **Academic Pages** Jekyll-based GitHub Pages template for academic personal websites. It's a static site generator that transforms Markdown files and YAML data into HTML pages hosted on GitHub Pages.

**Technology Stack:**
- Jekyll (static site generator)
- Ruby/Bundler (dependency management)
- Liquid templates (templating)
- Sass/SCSS (styling)
- Minimal JavaScript (jQuery, plugins)
- Kramdown (Markdown processor)

**Template Origin:** Forked from Minimal Mistakes Jekyll Theme

## Essential Commands

### Local Development

**Using Native Ruby/Jekyll:**
```bash
# Install dependencies (first time setup)
bundle install

# If you get permission errors, install gems locally:
bundle config set --local path 'vendor/bundle'
bundle install

# Serve the site locally (auto-rebuilds on changes)
jekyll serve -l -H localhost

# Alternative (ensures specific dependencies)
bundle exec jekyll serve -l -H localhost
```

The site will be available at `http://localhost:4000`

**Using Docker:**
```bash
# Set permissions and run
chmod -R 777 .
docker compose up
```

**Using Dev Container (VS Code):**
- Open in VS Code
- F1 → "Dev Container: Reopen in Container"
- Site automatically hosted at `http://localhost:4000`

### JavaScript Build

```bash
# Install npm dependencies
npm install

# Build minified JavaScript
npm run build:js
# or
npm run uglify

# Watch for JavaScript changes
npm run watch:js
```

### Content Generation

**Generate publications/talks from TSV:**
```bash
# Using Python scripts
cd markdown_generator
python publications.py
python talks.py

# Using Jupyter notebooks (preferred for documentation)
jupyter notebook publications.ipynb
jupyter notebook talks.ipynb
```

**Generate talk map:**
```bash
# Creates geographic visualization of talk locations
python talkmap.py

# Or use Jupyter notebook
jupyter notebook talkmap.ipynb
```

**Update CV JSON from Markdown:**
```bash
./scripts/update_cv_json.sh
```

## Project Structure

### Directory Organization

```
.
├── _config.yml              # Main site configuration (CRITICAL FILE)
├── _data/                   # YAML data files
│   ├── navigation.yml       # Header navigation menu
│   ├── authors.yml          # Author information
│   ├── ui-text.yml          # UI text strings
│   └── cv.json              # JSON CV data
├── _includes/               # Reusable HTML/Liquid components
│   ├── head.html
│   ├── footer.html
│   ├── masthead.html
│   └── author-profile.html
├── _layouts/                # Page templates
│   ├── default.html         # Base layout
│   ├── single.html          # Single post/page
│   ├── archive.html         # Collection listing
│   ├── talk.html            # Talk template
│   └── cv-layout.html       # CV template
├── _sass/                   # Sass/SCSS stylesheets
│   ├── _themes.scss         # Theme variables
│   └── _syntax.scss         # Code syntax highlighting
├── assets/                  # Compiled assets
│   ├── css/
│   ├── js/
│   └── fonts/
├── images/                  # Image assets
├── files/                   # PDF, slides, papers (publicly accessible)
├── _pages/                  # Standalone pages (about, CV, etc.)
├── _posts/                  # Blog posts
├── _publications/           # Publication entries
├── _talks/                  # Talk/presentation entries
├── _teaching/               # Teaching experience entries
├── _portfolio/              # Portfolio project entries
├── _drafts/                 # Draft content (not published)
├── markdown_generator/      # Scripts to generate markdown from TSV
└── scripts/                 # Utility scripts
```

### Collections

Jekyll collections are defined in `_config.yml` and correspond to directories:
- `_publications/` → `/publication/` URLs
- `_talks/` → `/talks/` URLs  
- `_teaching/` → `/teaching/` URLs
- `_portfolio/` → `/portfolio/` URLs
- `_posts/` → `/posts/` URLs

Each collection item is a markdown file with YAML front matter.

## Content Structure

### YAML Front Matter

All content files start with YAML front matter between `---` markers.

**Blog Posts** (`_posts/YYYY-MM-DD-title.md`):
```yaml
---
title: 'Blog Post Title'
date: 2024-01-15
permalink: /posts/2024/01/blog-post-title/
tags:
  - tag1
  - tag2
---
```

**Publications** (`_publications/YYYY-MM-DD-title.md`):
```yaml
---
title: "Paper Title"
collection: publications
category: conferences  # or manuscripts, books
permalink: /publication/2024-paper-title
excerpt: 'Brief description'
date: 2024-01-15
venue: 'Conference/Journal Name'
paperurl: 'http://example.com/paper.pdf'
citation: 'Your Name. (2024). "Paper Title." <i>Venue</i>. 1(1).'
---
```

**Talks** (`_talks/YYYY-MM-DD-title.md`):
```yaml
---
title: "Talk Title"
collection: talks
type: "Talk"  # or Conference proceedings, Tutorial, etc.
permalink: /talks/2024-talk-title
venue: "Conference/Institution Name"
date: 2024-01-15
location: "City, State, Country"
---
```

**Teaching** (`_teaching/YYYY-semester-title.md`):
```yaml
---
title: "Course Name"
collection: teaching
type: "Undergraduate course"  # or Graduate course
permalink: /teaching/2024-spring-course
venue: "University Name, Department"
date: 2024-01-15
location: "City, Country"
---
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jurgendn/jurgendn.github.io](https://github.com/jurgendn/jurgendn.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
