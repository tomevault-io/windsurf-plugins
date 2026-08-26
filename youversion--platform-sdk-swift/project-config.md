---
trigger: always_on
description: This is a public repo, living at https://github.com/youversion/platform-sdk-swift
---

# Agent instructions for `platform-sdk-swift`

This is a public repo, living at https://github.com/youversion/platform-sdk-swift

It provides an SDK for developers to use the YouVersion Platform APIs easily on Swift platforms e.g. iOS and iPadOS.

The SDK includes API access helpers, convenience classes and methods for working with Bible text, and SwiftUI views to make displaying scripture easy.

## Documentation

In addition to the README.md in this repo, more official documentation is at https://developers.youversion.com/sdks/swift

## Structure

This repo contains the SPM package YouVersionPlatform, which consists of:
- YouVersionPlatformCore: the core classes and supporting helpers for making API calls
- YouVersionPlatformUI: the base SwiftUI views for e.g. displaying Bible text with BibleTextView
- YouVersionPlatformReader: a fully-featured, drop-in Bible reader called BibleReaderView, built to look and work like the YouVersion Bible App. Includes many UI components such as Bible version pickers, Book/Chapter picker, footnote display, etc.

Tests are in the top-level Tests dir. The tests of YouVersionPlatformCore must run on environments e.g. Linux which don't have SwiftUI available.

## Required local checks

- Always run `swift test` and SwiftLint on code changes before finalizing work.

## Public API Stability

Source-breaking changes to the SDK's public interface are blocked by a CI job
(`.github/workflows/api-stability.yml`) that diffs the PR's API surface against
committed baselines under `.api-baseline/`. Additive changes pass; renames,
removals, and signature changes fail until the baseline is intentionally
updated as part of a major version bump.

To verify locally:

```bash
scripts/check-api-stability.sh check
```

When a breaking change is intentional (typically as part of a major version
bump), update the baselines and commit them in the same PR:

```bash
scripts/check-api-stability.sh update
```

The baseline files are JSON dumps from `swift-api-digester` covering
`YouVersionPlatformCore`, `YouVersionPlatformUI`, and `YouVersionPlatformReader`.

## SwiftLint

In a Linux container, run SwiftLint with SourceKit configured:

```bash
LINUX_SOURCEKIT_LIB_PATH=/root/.local/share/swiftly/toolchains/6.1.3/usr/lib swiftlint --strict
```

(If SwiftLint is not installed, get it from https://github.com/realm/SwiftLint/releases/latest/download/swiftlint_linux_amd64.zip or similar.)

### Testing Strategy
- Unit tests for core functionality

### Code Style
- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#specification) for commit messages (enforced in CI by the `Commit Lint` workflow — no local hook required)
- Protocol-oriented programming patterns
- Extensive use of extensions for code organization

## Common Workflows

### Git Branching Process

**⚠️ IMPORTANT: Every change goes on a branch, every branch is named after its Jira ticket, and every merge into `main` goes through a pull request. No direct edits or pushes to `main`.**

**Branch naming**: `<JIRA-TICKET>-<kebab-description>`

- Examples: `YPE-2293-swift-sdk-add-x-yvp-sdk-http-header-for-version-reporting`, `BA-1204-plans-update`, `BA-5678-bibles-cache-cleanup`
- The ticket prefix comes first; no initials prefix, no `feature/` prefix.
- Every branch — including doc-only edits, tooling changes, and small fixes — must have a Jira ticket and follow this pattern. If there's no ticket, create one before starting work.

**Standard workflow**:
1. Create the branch from `main`.
2. Make changes on the branch.
3. Open a PR back to `main`. PR title matches the first line of the commit message.

**Feature branches** (for large tasks or risky changes spanning multiple sub-tickets):
1. Create the feature branch from `main` using the parent epic's ticket: `<EPIC-TICKET>-<kebab-description>` (e.g., `YPE-1900-offline-search`).
2. Create task branches off the feature branch using each sub-ticket: `<TASK-TICKET>-<kebab-description>`.
3. Open PRs from task branches targeting the feature branch.
4. Open a final PR from the feature branch to `main` once the feature is complete.

**Updating feature branches with changes from `main`**:
- Merge `main` into the feature branch first.
- Then merge the updated feature branch into the task branch.
- Never merge `main` directly into a task branch.

### Adding New Features
1. Identify the appropriate internal framework or create new module
2. Follow existing architectural patterns (MVC/MVVM)
3. Add unit tests for new functionality
4. Update localization if needed
5. Run `swiftlint` to ensure code style compliance

## Important Tips
- Never commit a real `appKey` to the remote. In `Examples/SampleApp/SampleApp.swift`, keep the tracked placeholder `<#Your App Key#>` and leave your real key as an unstaged local change.
- Use GitHub to create pull requests (PRs).
- PR titles should always be the same as the first line of the commit message.

## Code style references

When writing or reviewing code in this repo, load the relevant skill:

- **`audit-swift`** (`.claude/skills/audit-swift/SKILL.md`) — project Swift conventions: access control, async/await, code organization, idioms, formatting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youversion/platform-sdk-swift](https://github.com/youversion/platform-sdk-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
