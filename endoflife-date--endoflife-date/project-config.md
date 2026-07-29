---
trigger: always_on
description: This is a Jekyll-based static site that tracks End of Life dates and support lifecycles for various products. The site is built and deployed to Netlify.
---

# endoflife.date Copilot Instructions

This is a Jekyll-based static site that tracks End of Life dates and support lifecycles for various products. The site is built and deployed to Netlify.

## Build, Test, and Lint Commands

### Development Server

```bash
# Install dependencies (first time only)
bundle install

# Run site locally
bundle exec jekyll serve --host localhost --port 4000
# Browse to http://localhost:4000
```

### Building

```bash
# Build the site (output to _site/)
bundle exec jekyll build
```

### Linting

```bash
# Lint a single product file
bin/lint-product.sh products/<product>.md

# Lint all markdown files (run by CI)
npx markdownlint-cli2@latest '**/*.md' '!node_modules' '!vendor'

# Format all files (run by CI)
npx prettier@latest --write .

# Validate formatting (run by CI)
npx prettier@latest --check .
```

### Testing API

```bash
# First tab - run Jekyll
bundle exec jekyll serve

# Second tab - run wiretap for API testing
npx @pb33f/wiretap@latest -s http://localhost:4000/docs/api/v1/openapi.yml -u http://localhost:4000
# Then open http://localhost:9091/ in browser
```

## Architecture

### Product Data Files

- **Product definitions**: Each product is a markdown file in `products/` with YAML frontmatter
- **Schema validation**: Product files validate against `product-schema.json`
- **Categories**: Products are categorized as: `app`, `db`, `device`, `framework`, `lang`, `library`, `os`, `server-app`, `service`, `standard`
- **Data structure**: Each product contains:
  - Metadata (title, permalink, category, tags, icons)
  - Column configuration (which columns to display: eol, eoas, eoes, discontinued, etc.)
  - Custom fields for product-specific data
  - Releases array (sorted newest first)
  - Markdown content after `---` (description and additional info)

### API Generation

- **Plugin-based**: Custom Jekyll plugins in `_plugins/` generate JSON API files during build
- **Main API plugin**: `_plugins/generate-api-v1.rb` creates the `/api/v1/` endpoints
- **OpenAPI spec**: `api_v1/openapi.yml` defines the API schema

### Automation

- **Release data**: Separate repo [`release-data`](https://github.com/endoflife-date/release-data) contains automation scripts
- **Auto-update**: Products with `auto:` configuration get releases automatically updated via:
  - Git tags (GitHub/GitLab repos)
  - Docker Hub
  - npm registry
  - DistroWatch
  - Maven Central
  - Custom scripts
- **CI workflow**: `.github/workflows/auto-merge-release-updates.yml` automatically merges release updates

### Theme and Layout

- **Base theme**: Built on [Just the Docs](https://github.com/just-the-docs/just-the-docs) Jekyll theme
- **Product layout**: `_layouts/product.html` renders individual product pages
- **Partials**: `_includes/` contains reusable components
- **Styling**: `_sass/` contains SASS files

## Key Conventions

### Product Files

1. **Naming**: Filename is `productname.md` (lowercase, dashes for spaces)
2. **Frontmatter only**: Product files are YAML frontmatter with markdown content below
3. **Frontmatter order** (blank line between sections):
   - Product info: `title`, `category`, `tags`, `iconSlug`, `permalink`, `alternate_urls`, `versionCommand`, `releasePolicyLink`, `changelogTemplate`
   - Formatting: `releaseLabel`, `LTSLabel`, `eolColumn`, `eoasColumn`, `releaseDateColumn`, `discontinuedColumn`, `eoesColumn`, etc.
   - Identifiers: `identifiers`
   - Auto-update: `auto`
   - Releases: `releases` (each release separated by blank line)
4. **UTC dates**: Use UTC timezone for all dates
5. **Date format**: Use `YYYY-MM-DD` format (unquoted for actual dates - never quote dates)
6. **Version strings**: Always quote version numbers like `"1.2.3"`
7. **Version ranges**: Use space-surrounded dash: `"2 - 5"`
8. **Version lists**: Comma and space separated: `"2, 4 - 7, 9"`
9. **Release cycles**: Use format like `"1.2"` (major.minor), lowercase, no "v" prefix
10. **Release ordering**: Releases must be sorted newest to oldest (each separated by blank line)
11. **Stable only**: Don't add RC/Alpha/Beta/Nightly releases
12. **Boolean dates**: Use boolean `true`/`false` when exact date is unknown
13. **changelogTemplate**: Keep on one line, use double quotes if containing liquid expressions

### Product Content

1. **First paragraph**: Must be a blockquote with product name linked to official site
2. **Description scope**: Keep product description limited to first blockquote only
3. **Line length**: Try to keep at 100 characters maximum
4. **Links**: No link reference definitions except for repeated links
5. **Acronyms**:
   - Explain acronyms if not obvious or part of product name
   - Use `*[ACRONYM]: Full Name` syntax at end of file (not `<abbr>` tags)
   - This avoids repeating definitions
6. **Summary**: Follow with brief release/EOL policy summary
7. **Focus**: Answer key questions readers have:
   - Which versions are supported?
   - Is my version supported?
   - Which version am I running? (via `versionCommand`)
   - How long until I have to upgrade?
   - When is the next release? (if feasible)
   - What does "supported" mean?
8. **Tone and tense**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [endoflife-date/endoflife.date](https://github.com/endoflife-date/endoflife.date) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
