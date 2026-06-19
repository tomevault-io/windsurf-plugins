---
trigger: always_on
description: Draft and publish GitHub release notes from actual git diffs and tags using gh, or turn the same release evidence into docs-backed article pages. Use when Codex needs to create, revise, or verify release notes for a GitHub release, especially when the notes must be based on real code changes, commit ranges, touched files, validation results, or a first release with no previous tag. Also use it when the user wants release articles written from GitHub release material or repository changes. When t
---


# GitHub Release Notes

## Overview

Create release notes from repository evidence, not commit subjects alone.

Use this skill to:

- draft release notes before publishing a tag
- rewrite thin or inaccurate existing GitHub release bodies
- create or update GitHub releases with `gh release create` or `gh release edit`
- handle first releases where there is no previous tag
- draft docs-backed article pages from the same release evidence when the user wants a blog or announcement post
- create a companion walkthrough article by default when publishing a release from a repository that already has a docs surface
- derive a new versioned release header SVG when the repository already has an earlier release header asset
- derive a new versioned release header SVG by default when the repository already ships suitable reusable SVG branding such as `assets/icon.svg`, `assets/logo.svg`, or a branded `assets/social-card.svg`, the release would benefit from a hero image, and there is no earlier versioned release header asset yet
- validate every candidate SVG before reusing existing header art or deriving new release header assets from repo branding
- mirror release notes into repository docs by default when the target repository already publishes docs
- keep release collateral, README, and primary operator docs aligned with code-backed shipped behavior
- require a release QA inventory artifact and validate it before closing the task
- keep release notes grounded in actual shipped behavior

## Prerequisites

- access to a local clone of the target repository
- `git` available in the shell
- `gh` installed and authenticated if publishing is requested
- permission to inspect diffs, tags, and release state

## Default Workflow

1. Read the target repository `README` if one exists and inspect the tag or release state.
   - Run `gh auth status` if the user expects you to publish or edit a GitHub release.
   - Fetch tags first when the local clone may be stale.
2. Determine the comparison range.
   - If the target tag has a previous tag, compare from that previous tag to the target tag.
   - If there is no previous tag, treat the release as an initial release and cover the full shipped history explicitly.
   - Use an explicit base tag when the release line is non-linear or when backfilled tags make auto-detection ambiguous.
3. Run the bundled collector first.
   - `powershell -ExecutionPolicy Bypass -File ./scripts/collect-release-context.ps1 -Tag v0.1.0`
   - If the tag exists only on GitHub and not in the local clone yet, fetch tags first or use `-Target <commit-or-branch>`.
   - Use `-BaseTag <tag>` to override the automatically detected previous tag.
   - Read the output, then inspect the actual diffs for the high-impact files and commits.
4. Review implementation diffs, not just summaries.
   - Read the changed file list and diff stats first.
   - Always inspect new or heavily changed scripts, workflows, fixtures, docs, and user-facing assets.
   - Search for existing release header assets such as `assets/release-header-v0.2.0.svg`, `docs/public/.../release-header-v0.2.0.svg`, or similar versioned SVGs before deciding whether to create a new header image.
   - If there is no versioned release header asset yet, search for reusable SVG branding such as `assets/icon.svg`, `assets/logo.svg`, `assets/social-card.svg`, or equivalent repo branding and treat that as the default seed for a new `release-header-v*.svg` only when the branding is suitable for a release hero image.
   - Validate every candidate SVG you might reuse with `powershell -ExecutionPolicy Bypass -File ./scripts/verify-svg-assets.ps1 -RepoPath . -Path <svg-paths>` before treating it as a reusable seed. Broken XML, missing `<svg>` roots, unresolved internal id references, or SVGs without scalable dimensions are not valid release-header inputs.
   - Use `git show` on major commits and touched files until you can name concrete capabilities added.
   - For implementation-sensitive claims such as model selection, retry/backoff, routing, defaults, environment variables, telemetry surfaces, or command behavior, inspect the actual code paths and tests that implement them.
   - When those claims depend on configuration or runtime wiring, read the config readers, command entrypoints, runtime call sites, and operator-visible output formatting instead of relying on one file in isolation.
   - If a behavior applies only to one command, service, embed, or deployment mode, keep the wording scoped to that exact surface instead of generalizing it.
5. Draft the requested output in the user's requested language.
   - Open with release scope and whether it is an initial release.
   - Group the material by user-visible capabilities and implementation areas, not by raw commit count.
   - Mention validation only if you actually ran it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunwood-ai-labs/gh-release-notes-skill](https://github.com/Sunwood-ai-labs/gh-release-notes-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
