---
trigger: always_on
description: Upgrade Helper is a static web tool that helps developers upgrade React Native applications by showing the full diff between two versions and layering in curated guidance.
---

# Upgrade Helper — Agent Guide

## Overview

Upgrade Helper is a static web tool that helps developers upgrade React Native applications by showing the full diff between two versions and layering in curated guidance.

This repository is valuable because it combines:

- upstream template diffs
- release-specific upgrade notes
- inline comments on important files
- binary file download support
- a simple UI that helps users track upgrade progress

Upgrade Helper is both:

- a trust-sensitive user-facing tool for React Native upgrades
- an open-source repository that requires steady, low-friction maintenance

Treat both responsibilities seriously. Correctness, stability, and clarity matter more than novelty.

---

## Primary counterpart

Your main human counterpart will often be one of the maintainers of this project.

Optimize for reducing maintainer burden while preserving project quality, trust, and stability.

That means you should be useful not only for product and code changes, but also for the ongoing work of maintaining an open-source repository, including triage, reproduction, documentation, test hygiene, CI/workflow fixes, and contributor-facing improvements.

When context is incomplete, do as much of the legwork as possible before asking for help:

- inspect the relevant code and docs
- reproduce the issue if possible
- identify affected files and workflows
- summarize likely causes
- propose the smallest safe next step

Prefer giving maintainers ready-to-use output:

- concise issue summaries
- reproduction notes
- draft PR descriptions
- documentation updates
- focused fix plans
- validation results

---

## Maintainer support scope

This agent supports maintainers with both code work and repository maintenance work.

Treat the following as first-class tasks:

- issue triage and bug reproduction
- narrowing down regressions
- reviewing contributor changes for risk, correctness, and missing validation
- improving README, contributing docs, and contributor guidance
- updating release-specific upgrade content under `src/releases/`
- maintaining tests, fixtures, mocks, and CI/workflow health
- identifying small, safe modernization opportunities in an aging codebase
- preparing clear summaries for maintainers to use in issues, PRs, or discussions

When helping with repository maintenance:

- prefer reducing ambiguity for maintainers
- prefer small, reviewable changes
- surface risks, follow-ups, and missing information clearly
- do not assume maintainers want a broad refactor when a targeted fix will do
- separate immediate fixes from longer-term cleanup ideas

---

## Product priorities

1. Preserve trust in the upgrade guidance
2. Keep user-facing behavior stable and predictable
3. Prefer small, reversible maintenance changes
4. Keep the app static and client-side
5. Keep contributor and maintainer workflows lightweight and clear

---

## Technical reality

This is an older but functioning frontend codebase.

Current stack:

- React 18
- Create React App via `react-app-rewired`
- TypeScript with `strict: true` and `allowJs: true`
- Emotion for styling
- Ant Design for UI components
- Framer Motion for transitions
- Yarn 1
- GitHub Pages deployment

Compatibility notes:

- `.node-version` and GitHub Actions currently target Node 16
- `mise.toml` declares Node 22 locally
- do not assume the repo is ready for Node-22-only changes unless you are explicitly modernizing the toolchain
- use `yarn`, not npm or bun, unless explicitly asked to change package management

This codebase should be treated as legacy-but-live: stable, useful, and worth improving carefully.

---

## Read first

Before making significant changes, read:

- `README.md`
- `CONTRIBUTING.md`
- `package.json`
- `src/components/pages/Home.tsx`
- `src/utils.ts`
- `src/releases/index.js`
- `src/releases/types.d.ts`

If touching UI and diff rendering, also read:

- `src/components/common/DiffViewer.tsx`
- `src/components/common/Diff/DiffSection.tsx`
- `src/components/common/Diff/DiffHeader.tsx`
- `src/__tests__/Home.e2e.spec.ts`

If touching release guidance, study an existing release file first, for example:

- `src/releases/react-native/0.60.tsx`
- `src/releases/react-native/0.77.tsx`

If touching maintainer or contributor workflows, also read:

- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/workflows/push.yml`
- `.github/workflows/deploy.yml`

Then read the files directly relevant to your task.

---

## Architecture map

- `src/components/pages/Home.tsx`
  - top-level page orchestration
  - theme toggle
  - URL state
  - settings and version selectors

- `src/hooks/fetch-release-versions.ts`
  - fetches available release versions from upstream repos

- `src/hooks/fetch-diff.ts`
  - fetches raw diff files and parses them for rendering

- `src/utils.ts`
  - central URL builders
  - changelog URL generation
  - version filtering
  - app name and package replacement
  - other shared logic

- `src/releases/`
  - curated release notes and inline upgrade comments
  - this is part of the product, not incidental content

- `src/theme/`
  - light/dark theme tokens

- `src/utils/test-utils.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [react-native-community/upgrade-helper](https://github.com/react-native-community/upgrade-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
