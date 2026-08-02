---
trigger: always_on
description: This is the official user documentation for [WLED](https://github.com/wled/WLED), published at [kno.wled.ge](https://kno.wled.ge) using [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).
---

# AGENTS.md

## Repository Overview

This is the official user documentation for [WLED](https://github.com/wled/WLED), published at [kno.wled.ge](https://kno.wled.ge) using [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

All documentation is written in **English**. Contributors improve docs by editing Markdown files and submitting pull requests.

[WLED](https://github.com/wled/WLED) is an open source firmware - written mainly in C++ using platformIO and node.js tooling - for ESP32/ESP8266 microcontrollers controlling addressable LEDs, with a web UI and various networking APIs for custom integrations. WLED is licensed under EUPL-1.2.

---

## Repository Structure

```
WLED-Docs/
├── docs/                   # All documentation source files
│   ├── index.md            # Landing page
│   ├── assets/             # Images, stylesheets, JS, MkDocs overrides
│   │   └── images/content/ # Preferred location for inline images
│   ├── basics/             # Getting started, installation, compatibility, FAQ
│   ├── features/           # WLED feature pages (effects, segments, presets, etc.)
│   ├── interfaces/         # API and protocol docs (JSON API, MQTT, E1.31, etc.)
│   ├── advanced/           # Wiring, compilation, custom features, etc.
│   └── about/              # Contributors, roadmap, privacy policy
├── mkdocs.yml              # Site config, navigation, theme settings
└── README.md               # Contributor guide
```

---

## Page Conventions

Every documentation page must begin with a YAML front matter block:

```yaml
---
title: Page Title
hide:
  # - navigation
  # - toc
---
```

- `title` is required and must match the nav entry in `mkdocs.yml`.
- `navigation` and `toc` are commented out by default (i.e., both are shown). Only hide them when there is a clear design reason (e.g., the landing page hides both).

---

## Navigation Registration

Every new page **must** also be registered in the `nav:` section of `mkdocs.yml`. Pages not listed there will not appear in the site navigation.

```yaml
nav:
  - Section Name:
    - Page Title: section/filename.md
```

Indentation in `mkdocs.yml` is significant.

---

## Markdown Style

- **Headings**: Use `##` for top-level sections within a page (`#` is optional, as it may conflict with the page title). Use `###` and `####` for sub-sections.
- **Admonitions**: Use MkDocs Material admonitions (`!!! info`, `!!! tip`, `!!! warning`, `!!! danger`) for callouts. These are rendered as styled boxes on the site.
- **Tables**: Use standard GFM pipe tables.
- **Code blocks**: Use fenced code blocks with a language hint (e.g., ` ```yaml `).
- **Bold/Italic**: Use `**bold**` for important terms. Use `_italic_` sparingly.
- **Line breaks**: A trailing two-space line break (or `<br />`) is used to force a line break within a paragraph where needed.
- **Internal links**: Use root-relative paths without the `.md` extension, e.g. `[Segments](/features/segments)`.
- **External links**: Use standard Markdown link syntax. Prefer linking to stable, canonical URLs.

---

## Images

- Store images in `docs/assets/images/content/` whenever possible. Avoid external image hosting — the external resource could disappear at any time.
- Reference images using a path relative to the page or a root-relative path, e.g.:
  ```markdown
  ![Alt text](../assets/images/content/my-image.png)
  ```
- Animated GIFs hosted on `raw.githubusercontent.com/scottrbailey/WLED-Utils/` are used extensively in the Effects table and are acceptable for that page.

---

## PR Review Checklist

When reviewing a pull request, verify the following:

### Content

- [ ] All text is in **English**.
- [ ] Content is accurate and consistent with the current WLED feature set.
- [ ] No duplicate content — check whether similar information already exists on another page and cross-link instead.
- [ ] Links are valid — no broken internal or external links.
- [ ] Images are stored in `docs/assets/images/content/` (not external hosting) unless they are GIF previews from the known WLED-Utils repository.

### Formatting

- [ ] Front matter block (`---`) is present and includes a `title` field.
- [ ] Page title in front matter matches the entry in `mkdocs.yml`.
- [ ] Heading hierarchy is correct (`##` → `###` → `####`, no skipping levels).
- [ ] Admonitions use the correct type (`info`, `tip`, `warning`, `danger`) for the context.
- [ ] Tables are properly formatted with pipe characters.
- [ ] Code blocks use language identifiers.
- [ ] No unintentional trailing whitespace (two trailing spaces are allowed for forced line breaks).

### Writing Style & Readability

- [ ] Tone is **informal, friendly, and inviting** — avoid overly formal or academic language.
- [ ] Title case: Use title case for page title and section headings; articles and short prepositions stay lowercase.
- [ ] Language is **simple and clear** — short sentences, common words, no unexplained jargon — so non-native English speakers can easily follow.
- [ ] Logic is easy to follow, examples are kept simple.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wled/WLED-Docs](https://github.com/wled/WLED-Docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
