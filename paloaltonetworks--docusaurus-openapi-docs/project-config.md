---
trigger: always_on
description: This repository hosts the **docusaurus-plugin-openapi-docs** and **docusaurus-theme-openapi-docs** packages. The plugin converts OpenAPI specs to MDX and the theme provides React components for rendering those docs. The `demo` directory shows them working together.
---

# Contributor Guide

This repository hosts the **docusaurus-plugin-openapi-docs** and **docusaurus-theme-openapi-docs** packages. The plugin converts OpenAPI specs to MDX and the theme provides React components for rendering those docs. The `demo` directory shows them working together.

```
root
├─ packages
│  ├─ docusaurus-plugin-openapi-docs   # generates MDX from OpenAPI specs
│  └─ docusaurus-theme-openapi-docs    # theme components for API docs
├─ demo                                # example site using the plugin and theme
└─ scripts                             # release and helper scripts
```

The plugin and theme are typically used together:

```
OpenAPI spec ──▶ plugin ──▶ generated MDX ──▶ docs plugin ──▶ theme ──▶ website
```

## Developer Quick Start

Steps from the top-level `README.md` for getting started:

```bash
 git clone https://github.com/<your account>/docusaurus-openapi-docs.git
 cd docusaurus-openapi-docs
yarn
yarn build-packages
yarn watch:demo
```

## Contributing Tips

See `CONTRIBUTING.md` for the full guide. Use clear commit messages so reviewers can understand what each commit does.

## Required Checks

Run the following before committing any code changes (except documentation or comment-only updates):

```bash
yarn lint
yarn test
```

## Other Common Tasks

- `yarn` – install dependencies
- `yarn build-packages` – compile all packages

## Versioning and Releases

This project uses [semantic versioning](https://semver.org/) and Lerna for package management. Releases are triggered automatically when version changes are merged to `main`.

### Version Types

| Bump Type    | Example                         | When to Use                             |
| ------------ | ------------------------------- | --------------------------------------- |
| `patch`      | `4.5.0` → `4.5.1`               | Bug fixes, minor documentation updates  |
| `minor`      | `4.5.1` → `4.6.0`               | New features, non-breaking enhancements |
| `major`      | `4.6.0` → `5.0.0`               | Breaking changes                        |
| `preminor`   | `4.5.1` → `4.6.0-rc.0`          | Pre-release for upcoming minor version  |
| `prerelease` | `4.6.0-rc.0` → `4.6.0-rc.1`     | Iterate on existing pre-release         |
| `graduate`   | `4.6.0-rc.5` → `4.6.0`          | Promote pre-release to stable           |
| `betamajor`  | `4.5.1` → `5.0.0-beta.0`        | Start a new beta major version          |
| `betapatch`  | `5.0.0-beta.0` → `5.0.0-beta.1` | Iterate on beta version                 |

### Preparing a Release

#### Step 1: Bump the Version

Run the version command with the appropriate bump type:

```bash
yarn release:version <bump>
```

This updates `lerna.json` and all package versions. The version is stored in `lerna.json` and propagated to all packages.

#### Step 2: Generate the Changelog

```bash
yarn release:changelog
```

This outputs a changelog template comparing commits between the latest tag and `main`. Copy the output and prepend it to `CHANGELOG.md`.

**Changelog format:**

The changelog uses a categorized format inspired by [Docusaurus](https://github.com/facebook/docusaurus/blob/main/CHANGELOG.md). Each release includes a high-level summary followed by categorized changes with emoji headers.

```markdown
## X.Y.Z (YYYY-MM-DD)

Brief summary of the most significant changes in this release (1-3 sentences).

- Bullet points highlighting major user-facing features or fixes

#### :rocket: New Feature

- feat: description of feature ([#123](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs/pull/123))

#### :bug: Bug Fix

- fix: description of fix ([#124](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs/pull/124))

#### :house: Refactoring

- refactor: description ([#125](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs/pull/125))

#### :memo: Documentation

- docs: description ([#126](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs/pull/126))

#### :robot: Dependencies

- chore(deps): bump package from X to Y ([#127](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs/pull/127))

#### Committers: N

- Name or username
```

**Category headers (include only sections with changes):**

| Emoji           | Category      | Commit Prefix   |
| --------------- | ------------- | --------------- |
| :rocket:        | New Feature   | `feat`          |
| :bug:           | Bug Fix       | `fix`, `bugfix` |
| :running_woman: | Performance   | `perf`          |
| :nail_care:     | Polish        | `style`         |
| :house:         | Refactoring   | `refactor`      |
| :memo:          | Documentation | `docs`          |
| :test_tube:     | Testing       | `test`          |
| :robot:         | Dependencies  | `chore(deps)`   |
| :wrench:        | Maintenance   | `chore`         |

**Guidelines for changelog entries:**

- Replace `TODO: Add high-level summary` with a concise summary of user-facing changes
- Use ISO date format: `YYYY-MM-DD`
- Remove internal commits (CI changes, workflow updates) that don't affect users

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaloAltoNetworks/docusaurus-openapi-docs](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
