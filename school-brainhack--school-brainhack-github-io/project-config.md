---
trigger: always_on
description: **Brainhack School** is a 4-week distributed neuroscience and open-science training event running simultaneously at multiple global sites. Participants complete structured training modules (Week 1) then work on collaborative data-science projects (Weeks 2–4).
---

# CLAUDE.md — Brainhack School Website

## Project Overview

**Brainhack School** is a 4-week distributed neuroscience and open-science training event running simultaneously at multiple global sites. Participants complete structured training modules (Week 1) then work on collaborative data-science projects (Weeks 2–4).

- **Live site:** https://school-brainhack.github.io
- **Dev docs:** https://school-brainhack.readthedocs.io
- **Stack:** Hugo static site + GitHub Pages deployment
- **Theme:** `hugo-universal-theme` (Git submodule at `themes/hugo-universal-theme`)
- **Hugo version:** 0.128.0 extended

---

## Local Development

```bash
# 1. Initialize submodules (required after first clone)
git submodule update --init --recursive --remote

# 2. Serve locally with drafts
hugo serve -D

# 3. Production build (run by CI)
hugo --minify
```

Site is served at `http://localhost:1313` by default.

---

## Repository Structure

```
config.yaml              # Main Hugo configuration
content/en/              # All site content (markdown)
  modules/               # Training module pages (~26 modules)
  project/               # Student project gallery (60+ projects)
  sites/                 # Distributed site hubs (toronto, polytechnique, criugm, singapore, taiwan)
  weeks/                 # Schedule/week pages
  schedule/              # Calendar output page
  coc/                   # Code of Conduct
  guide.md               # Getting Started guide
  project_guide.md       # Project submission guide
  register.md            # Registration page
data/en/                 # YAML-driven dynamic content
  instructors.yaml       # Global instructor/team list
  carousel/              # Homepage carousel slides (7 YAML files)
  testimonials/          # Quote testimonials (3 YAML files)
  clients/               # Sponsor/partner logos (2 YAML files)
  features/              # Features section (disabled; 6 YAML files)
layouts/                 # Custom Hugo templates (override theme)
  partials/              # Reusable components (nav, footer, carousel, instructors…)
  shortcodes/            # Custom shortcodes: gform, tabs, tab, class, content
  project/               # Project list + single page layouts
  modules/               # Module list + single page layouts
  sites/                 # Site hub layouts
static/img/              # All images
  logo_brainhack_2025.png
  carousel/              # Carousel slide images
  instructors/           # Instructor headshots
  testimonials/          # Testimonial avatars
  locations/             # Site hub images
  clients/               # Sponsor logos
i18n/en.yaml             # English UI strings
i18n/fr.yaml             # French UI strings
.github/workflows/hugo.yml  # CI/CD pipeline
```

---

## config.yaml Key Settings

| Setting | Value | Notes |
|---|---|---|
| `baseURL` | `https://school-brainhack.github.io` | |
| `theme` | `hugo-universal-theme` | Git submodule |
| `params.style` | `green` | Theme color |
| `params.logo` | `img/logo_brainhack_2025.png` | Update for new year |
| `params.logo_small` | `img/logo-small_2025.png` | |
| `params.carousel.enable` | `true` | |
| `params.instructors.enable` | `true` | |
| `params.testimonials.enable` | `true` | |
| `params.features.enable` | `false` | Currently disabled |
| `params.recent_posts.enable` | `false` | Currently disabled |
| Markdown unsafe HTML | `true` | HTML allowed in markdown |

**Main menu** (by weight): Home → Projects → Schedule → Sites → Register → Modules
**Dropdowns:** `documents` (Getting Started, Project Guide, CoC, Dev Docs) · `past` (2024–2018 archives)
**Topbar:** GitHub, Mastodon, email contact

**Custom taxonomies:**
- `tags` → filter projects and modules
- `names` → author/organizer names (used for filtering)

---

## Content Frontmatter Reference

### Project page — `content/en/project/<slug>/index.md`

```yaml
type: "project"
date: "YYYY-MM-DD"
title: "Project Title"
names: [Author1, Author2]           # must match instructor list if applicable
github_repo: "https://github.com/..."
website: ""                          # optional project website
tags: [tag1, tag2]                   # LOWERCASE only
summary: "~75 word description"
image: "cover.png"                   # optional; file must exist in same directory
```

### Module page — `content/en/modules/<slug>/index.md`

```yaml
type: "modules"          # DO NOT change this field
title: "Module Title"
tags: [tag1, tag2]       # lowercase
summary: "~75 word description"
image: "cover.png"       # optional
```

### Site hub page — `content/en/sites/<slug>/index.md`

```yaml
type: "sites"
date: "YYYY-MM-DD"
title: "Institution Name, City, Country"
names: [Organizer1, Organizer2]
website: ""              # optional
summary: "~75 word description"
image: "location.png"
```

---

## Data Files — How to Update

### Add or update an instructor (`data/en/instructors.yaml`)

```yaml
- name: Full Name
  gh: github_username          # GitHub handle (no @)
  email: email@example.com
  website: https://...
  affiliation: Institution Name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [school-brainhack/school-brainhack.github.io](https://github.com/school-brainhack/school-brainhack.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
