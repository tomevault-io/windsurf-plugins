---
trigger: always_on
description: > **Note:** For comprehensive details on the build system, deployment infrastructure, and CI/CD workflows, see `BUILD-AND-DEPLOY.md`. This file is large, so read only specific sections as needed to conserve tokens.
---

# AGENTS.md — Pulumi website (Hugo): docs, blog, marketing, and example programs

---

## Build / Test / Lint Workflow

> **Note:** For comprehensive details on the build system, deployment infrastructure, and CI/CD workflows, see `BUILD-AND-DEPLOY.md`. This file is large, so read only specific sections as needed to conserve tokens.

> **Never push directly to `master`.** Always create a branch and open a PR. Direct pushes to `master` bypass review and CI checks. (Server-side branch protection is planned but not yet in place.)

Agents must use these exact commands:

- Install deps: `make ensure`
- Build site: `make build`
- Serve locally on port 1313 (accessible with curl):  
  - Normal: `make serve` 
  - With asset rebuilds: `make serve-all`
- Lint: `make lint` (must pass before commit/merge)
- Lint prose: `make lint-prose` (Vale; nags, never blocks. Also surfaces in pinned PR reviews.)
- Format: `make format`
- Run all tests: `make test`
- Run specific program test:  
  `ONLY_TEST="program-name" ./scripts/programs/test.sh`
- Fix trailing spaces:  
  `sed -i '' 's/[[:space:]]*$//' file1.md file2.md ...`

Do not substitute other tools or commands, or change `package.json` to use pnpm (Yarn/npm only).

---

## Code & Content Rules

For all content files, follow `STYLE-GUIDE.md`. If a rule is not covered there, fall back to the [Google Developer Documentation Style Guide](https://developers.google.com/style). Do not invent new style conventions; ask for clarification if something is ambiguous.

Meta files like this one, `BUILD-AND-DEPLOY.md`, and agent instruction/skill files (e.g., `.claude/commands/*.md`) are exempt from formatting rules (heading case, trailing newlines, etc.).

For all content files (docs, blogs, tutorials, etc.):

- **Markdown**: Must always end with a newline.
- **Headings**:  
  - H1 = Title Case  
  - H2+ = Sentence case
- **TypeScript/JavaScript**: Must follow `tsconfig.json` settings. No comments unless explicitly requested.
- **TypeScript program files** (`static/programs/`): Use hand-written constructor style — resource name and opening `{` on the same line, `}, {` inline when an opts argument follows:
  ```typescript
  const r = new SomeResource("name", {
      prop: value,
  }, { 
      provider: p,
  });
  ```
  Do NOT use Prettier's multi-arg style where name, props, and opts are each on separate indented lines.
- **File Placement**:  
  - Docs go under `content/docs/...`
  - Blog posts go under `content/blog/...`
  - Other content goes into appropriate `content/...` subdirectory
  - Code examples go under `/static/programs` with a language suffix in the filename.  
  - Mirror the structure of existing content; do not invent new layouts.
- **Includes**: Use Hugo shortcodes for shared content, never raw Markdown copy-paste.  
- **Naming**: Use lowercase for non-proper nouns (e.g. “stack,” not “Stack”).  
- **Ordered Lists**: Every item begins with `1.` to minimize diff noise.
- **Diagrams**: Prefer Mermaid diagrams over ASCII art. The site renders Mermaid natively via a Hugo code block hook (`layouts/_default/_markup/render-codeblock-mermaid.html`). Use ` ```mermaid ` fenced code blocks. See [Mermaid docs](https://mermaid.js.org/) for syntax.
- **Images on template-driven pages**: Place new images for template-driven pages (homepage, product pages, event pages, case studies — anything rendered through `layouts/partials/template-partials/*`) under `assets/fingerprinted/`, mirroring the path you'd use under `static/`. The template partials route every `<img>` through `layouts/partials/fingerprinted-img.html`, which content-hashes filenames, converts rasters to WebP, and generates responsive `srcset`s. Frontmatter paths still look like `/images/foo.svg`; the partial resolves them. Missing assets cause a build panic, so there is no silent fallback. `meta_image` and assets used by non-template layouts can stay in `static/`.
- **Meta images**: `meta_image` is optional for `docs`, `tutorials`, `case-studies`, `what-is`, `migrate`, `partner`, `topics`, `events`, and `blog` pages. Leave it blank and `scripts/generate-meta-images.mjs` produces an on-brand social card at build time (resolved by `layouts/partials/meta-image-url.html`). A page-level `meta_image` always wins, but custom overrides are discouraged — the generated card covers virtually every case and stays on-brand automatically. For blog posts the card is built from the post title + `feature_image` (generate the feature image with `/blog-feature-image`, or label the PR `needs-design` for a designer-made one); a post's off-brand legacy meta image, if any, was renamed to `meta-legacy.png` and shows in a collapsed "Archived feature image" panel.
- **Spelling/Grammar**: Always correct errors. Use American English spelling.

---

## Moving and Deleting Files

**⚠️ SEO CRITICAL**: Missing aliases on moved files break search rankings and external links.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pulumi/docs](https://github.com/pulumi/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
