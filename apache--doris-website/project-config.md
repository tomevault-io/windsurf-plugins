---
trigger: always_on
description: This is the codebase for the Apache Doris website and documentation site, built with Docusaurus. The current English docs live under `docs/`, active versioned docs live under `versioned_docs/`, Chinese localized docs live under `i18n/zh-CN/docusaurus-plugin-content-docs/`, community docs live under `community/`, Chinese localized community docs live under `i18n/zh-CN/docusaurus-plugin-content-docs-community/`, blog posts live under `blog/`, and the site UI/configuration lives under `src/`, `stat
---

# AGENTS.md — Apache Doris Website

This is the codebase for the Apache Doris website and documentation site, built with Docusaurus. The current English docs live under `docs/`, active versioned docs live under `versioned_docs/`, Chinese localized docs live under `i18n/zh-CN/docusaurus-plugin-content-docs/`, community docs live under `community/`, Chinese localized community docs live under `i18n/zh-CN/docusaurus-plugin-content-docs-community/`, blog posts live under `blog/`, and the site UI/configuration lives under `src/`, `static/`, `config/`, `docusaurus.config.js`, `sidebars.ts`, `sidebarsCommunity.json`, `versioned_sidebars/`, and `versions.json`.

For review tasks, if the PR involves changes to the website's basic framework and display, you need to verify from the code level whether the changes are reasonable and comply with various front-end standards and best practices. If the PR is about submitting document content, you need to assess whether the content is reasonable and complete, ensuring it does not cause any confusion or ambiguity from the user's perspective. In summary, your principle is: first follow the standards mentioned in this document for the review, but the ultimate goal is to ensure a complete and seamless user experience with the documentation.

## Repository Conventions

- Follow the existing structure and writing style in the touched area. Keep terminology, heading hierarchy, front matter shape, and file placement consistent with neighboring files.
- Treat `sidebars.ts`, `sidebarsCommunity.json`, `versioned_sidebars/`, `versions.json`, and `docusaurus.config.js` as the authoritative sources for navigation, active versions, and routing. Do not rely on older README tree examples when they disagree with the current repository layout.
- Preserve stable URLs whenever possible. When moving, renaming, or deleting docs, review inbound links, sidebar references, redirects, and versioned or localized counterparts that may also need updates.
- The current active docs versions are defined in `versions.json`. In the current repository state they are `4.x`, `3.x`, `2.1`, and `current`, and many docs changes intentionally update the same topic across `docs/`, `versioned_docs/`, and the mirrored `i18n/zh-CN` trees together. Review missing mirror updates against the actual touched page set and nearby history, and treat them as issues when the repository context shows those mirrors are expected to stay aligned.
- Keep Docusaurus metadata correct. When editing docs or blog files, verify any relevant front matter fields such as `title`, `description`, `slug`, `keywords`, `sidebar_position`, and tags.
- Prefer targeted edits over broad rewrites. Avoid reformatting unrelated content, reordering large sections, or changing tone across a whole file unless the task explicitly requires it.
- When touching React, TypeScript, styling, or site configuration, preserve the existing visual language and implementation patterns already used in this repository.
- When touching community docs, remember that navigation is controlled separately by `sidebarsCommunity.json` and Chinese localized community content lives under `i18n/zh-CN/docusaurus-plugin-content-docs-community/`.

## Code Review

When conducting code review, including automated AI review and self-review, you must follow this file as the review guide. There is no separate review skill in this repository.

During review, you must individually provide conclusions for each applicable critical checkpoint below:

- What is the goal of the current change? Does the implementation actually satisfy that goal?
- Is the change as small, clear, and focused as possible?
- Does it preserve documentation information architecture correctly? Check affected current docs, active versioned docs, localized docs, blog/community pages, sidebars, and config references when applicable.
- If any page path, filename, heading anchor, slug, or sidebar item changed, are related links, redirects, and navigation references in `sidebars.ts`, `sidebarsCommunity.json`, `versioned_sidebars/`, `i18n/`, and config updated consistently?
- If the change touches Docusaurus config, React components, styling, or build scripts, does it match existing repository conventions and avoid breaking desktop or mobile behavior?
- What validation covers the change? State which checks are applicable and whether the change is sufficiently verified.
- Based on the code and content context, are there any other correctness, usability, accessibility, SEO, or maintainability issues?

Review emphasis by change type:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/doris-website](https://github.com/apache/doris-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
