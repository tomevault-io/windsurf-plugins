---
trigger: always_on
description: Pull Request (PR) Creation Guidelines. Use these rules when creating pull requests
---


# MetaMask Mobile Pull Request (PR) Creation Guidelines

These rules apply whenever creating PRs in the MetaMask Mobile repository or suggesting PR creation steps to others. Follow them to ensure consistent quality and smooth reviews.

## Core Principles

- Prefer small, focused PRs that are easy to review.
- Make PR titles and descriptions clear, specific, and actionable.
- Always use the repository PR template and complete every section.
- Ensure correct labels and branch naming for automation and triage.

## 1. PR Title Requirements

PR titles must follow Conventional Commits and clearly describe the change.

- Allowed types (with brief descriptions):
  - `feat:` For new features or significant enhancements.
  - `fix:` For bug fixes or corrections.
  - `docs:` For documentation changes only.
  - `style:` For code style changes (formatting, whitespace, etc.) that do not affect logic.
  - `refactor:` For code changes that neither fix a bug nor add a feature (e.g., code restructuring).
  - `test:` For adding or updating tests.
  - `chore:` For routine tasks, maintenance, or changes to build tools and dependencies.
  - `perf:` For performance improvements.
  - `ci:` For changes to CI configuration or scripts.
  - `build:` For changes affecting the build system or external dependencies.
  - `revert:` For reverting previous commits.
- Keep titles concise (ideally under ~72 characters) and specific.
- Use imperative mood after the type (e.g., "add", "fix", "refactor").

### Examples

| ✅ Good                                                    | ❌ Bad           |
| ---------------------------------------------------------- | ---------------- |
| `feat: add NFT gallery to collectibles tab`                | `Add some stuff` |
| `fix: resolve wallet connection timeout on cold start`     | `fixing bug`     |
| `refactor: extract transaction formatter into utility`     | `refactor code`  |
| `test: add e2e spec for onboarding SRP import flow`        | `tests`          |
| `docs: update contributing guide with design-system usage` | `update docs`    |
| `chore: bump react-native to 0.74.5`                       | `upgrade rn`     |

### Title DO / DON'T

- ✅ DO: Start with a valid type prefix and a short, descriptive phrase
- ✅ DO: Reference scope briefly in the phrase if useful (e.g., "collectibles", "onboarding")
- ❌ DON'T: Include issue numbers or long explanations in the title (put in description)
- ❌ DON'T: Use ambiguous words like "stuff", "things", "fixes" without context

## 2. PR Template Compliance

You must use the template at `.github/pull-request-template.md`. Fill out all sections thoroughly.

- **Description**: What changed and why. Include context, constraints, and trade-offs.
- **Changelog entry**: Provide a user-facing change summary in past participle (e.g., "Added", "Fixed"). If not user-facing, write `CHANGELOG entry: null`.
- **Related issues**: Use `Fixes: #NUMBER`, `Closes: #NUMBER`, or `Refs: #NUMBER` as appropriate.
- **Manual testing steps (Gherkin format)**: Provide reproducible steps reviewers/QA can follow.
- **Screenshots/Recordings**: Required for UI changes (before/after when relevant).
- **Pre-merge checklist**: Ensure all items are checked (lint/tests pass, docs updated, feature flags set, etc.).

### Gherkin Example

```gherkin
Scenario: Import an existing wallet via SRP on fresh install
  Given the app is freshly installed
  And I am on the Welcome screen
  When I tap "Import using Secret Recovery Phrase"
  And I enter a valid 12-word SRP
  And I set a new password
  Then I should land on the Home screen
  And I should see the default account with a balance
```

### Changelog Examples

```text
CHANGELOG entry: Added NFT gallery to Collectibles tab.
CHANGELOG entry: Fixed wallet connection timeout on cold start.
CHANGELOG entry: null
```

## 3. Required Labels and Assignment

### PR Assignment

- **Always assign the PR to yourself (the author)** immediately after creation.
- This ensures proper ownership tracking and notifications.

### Automatic Team Label Detection

When creating PRs, automatically detect and apply the correct team label based on:

1. **Author's GitHub team membership** - Check which MetaMask teams the author belongs to
2. **Available team labels in the repository** - Match against existing `team-*` labels
3. **Context awareness** - Consider files changed when author belongs to multiple teams

#### Generic Team Label Detection Process

```bash
# Step 1: Get author's MetaMask teams
USER_TEAMS=$(gh api user/teams --paginate | jq -r '.[] | select(.organization.login == "MetaMask") | .slug')

# Step 2: Get available team labels in the repository
REPO_TEAM_LABELS=$(gh label list --search "team-" --limit 100 | cut -f1)

# Step 3: Try to find a matching team label
# Common mappings to check:
# - Exact match: team-{github-team} (e.g., mobile-platform → team-mobile-platform)
# - Without suffix: design-system-engineers → team-design-system
# - With wallet prefix: wallet-ux → team-wallet-ux

# If no clear match is found, DO NOT add a team label
# It's better to have no team label than an incorrect one
```

### Required Labels

Apply labels to enable automation and proper routing. Some labels can block merging.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiloOweno/node-forge](https://github.com/MiloOweno/node-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
