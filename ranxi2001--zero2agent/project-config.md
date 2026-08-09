---
trigger: always_on
description: This repository owns the zero2Agent tutorial site at
---

# zero2Agent Repository Guide

## Purpose

This repository owns the zero2Agent tutorial site at
`https://onefly.top/zero2Agent/`. It is a Chinese-language, engineering-focused
learning path for developers building, evaluating, and interviewing for AI
Agent systems.

Keep repository changes focused on one coherent content, navigation, design,
or publishing outcome. Preserve unrelated user edits and check the worktree
before and after every task.

## Start Every Task

1. Run `git status --short --branch` and identify the exact files in scope.
2. Read the relevant repository Skill under `.agents/skills/` before editing.
3. Inspect the current module index, neighboring articles, and `_data/nav.yml`;
   examples or article counts in a Skill may be older than the repository.
4. For facts about fast-moving tools, APIs, models, prices, or projects, verify
   them against current primary sources and distinguish fact from inference.
5. Keep generated output, credentials, browser profiles, and scraped raw data
   out of Git.

## Source Of Truth

- `_data/nav.yml` drives the article header navigation, sidebar, and previous/
  next pager through `_layouts/default.html`.
- Each module directory owns its landing `index.md` and article directories.
- An article lives at `<module>/<NN>-<slug>/index.md`.
- `index.html` owns the homepage module cards, headline metrics, and primary
  entry links.
- `README.md` owns the public repository overview, module table, totals, and
  local-run instructions.
- `_config.yml` owns the Jekyll URL, base path, included content, and plugins.
- `_layouts/default.html` and `assets/` own the shared article shell and client
  behavior. Avoid article-specific fixes in shared code unless the behavior is
  intentionally site-wide.
- `.github/workflows/jekyll-gh-pages.yml` is the production build and deploy
  contract. A push to `main` publishes the site.

When these sources disagree, use executable site behavior and current files as
evidence, then update every affected summary or count in the same change.

## Repository Skills

`.agents/skills` points to the canonical `.claude/skills` directory so different
agents use the same project procedures. Read the selected `SKILL.md` completely
before using it.

- `new-article`: article structure and writing workflow.
- `new-module`: module creation and site integration.
- `content-review`: seven-dimension editorial review.
- `chinese-quotes-fix`: protected-region-aware Chinese quote checks and fixes.
- `classify-interview-questions`: distribute interview questions by dimension.
- `mermaid-check`: Mermaid 9.4.3 compatibility and validation.
- `drawio-skill`: editable diagrams and rendered exports.
- `scrape-nowcoder`: login-dependent interview-content collection; use only
  when the current request explicitly authorizes it.

Repository source takes precedence over stale inventories or paths inside a
Skill. Improve the canonical Skill when a repeated rule has genuinely changed.

## Content Contract

- Write primarily in Chinese; keep code, commands, paths, API names, and
  established technical terms in their native form when clearer.
- Lead with the engineering problem and why it matters before definitions.
- Explain system roles, constraints, failure modes, and tradeoffs. Do not present
  a framework as universally best or confuse a runnable demo with production
  readiness.
- Use concise sections, concrete examples, typed code blocks, and small tables
  where they improve understanding.
- Keep claims attributable. Preserve and extend `THIRD_PARTY_NOTICES.md` when a
  change materially depends on an external source.
- Do not include private interview data, credentials, tokens, personal account
  details, or unredacted logs.

New articles use this frontmatter shape unless the surrounding module has a
stricter current convention:

```yaml
---
layout: default
title: "文章标题"
description: "一句具体描述"
eyebrow: "Module Name / NN"
---
```

When adding, removing, renaming, or reordering an article, update all affected
surfaces together:

1. The article directory and frontmatter.
2. The module landing page's reading order and article table.
3. `_data/nav.yml` for header/sidebar/pager behavior.
4. `index.html` and `README.md` when module cards, counts, status, or links change.
5. Neighboring explicit next-article links, when present.

Use relative repository links ending in `index.html` for site content. Check
both link destination existence and `baseurl: /zero2Agent` behavior.

## Diagrams And Assets

- Prefer fenced `mermaid` blocks for diagrams, including architecture,
  relationship, and multi-stage flow diagrams. The site currently loads Mermaid
  9.4.3 with `htmlLabels: false`; follow `mermaid-check` for compatible labels
  and syntax.
- Do not use Draw.io by default. Use `drawio-skill` only when Mermaid cannot
  express the required layout reliably or when the user explicitly requests an
  editable Draw.io canvas. Keep the editable `.drawio` source with its rendered
  asset when this exception applies.
- Store durable images under the relevant article directory or `assets/images/`
  using descriptive names and alt text. Do not commit temporary renders.
- Visually inspect changed pages at desktop and mobile widths when layout,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ranxi2001/zero2Agent](https://github.com/ranxi2001/zero2Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
