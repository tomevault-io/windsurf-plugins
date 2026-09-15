---
trigger: always_on
description: Every new user-facing string must be defined in the project's string resource files, including
---

# Naviamp Agent Development Rules

## Translatable User-Facing Strings

Every new user-facing string must be defined in the project's string resource files, including
`strings.xml` and each maintained translation. Do not hardcode new UI copy in Kotlin or
platform-specific source files.

## Settings Export and Import

Every new or changed setting that is usable across platforms must be included in shared settings
export/import and sync. Verify round-trip preservation, defaults for older exports, and normalization
of imported values. Keep only genuinely device-specific settings local.

## Release Announcements

Whenever a new Naviamp release is pushed, create a GitHub Discussion in the **Announcements**
category that explains what is new, what changed, important fixes, and any upgrade or compatibility
notes. Feature branches and other unreleased work do not receive release announcements.

Release notes must compare the release with the previous public release, not narrate development on
the release branch. A platform or feature shipping for the first time is one complete new
capability; prerelease implementation fixes are not separate public changes.

Order announcements by product significance. Major launches lead the title, summary, and
highlights. Improvements and fixes to previously released behavior follow.

## In-App Release Changelog

Before tagging every Naviamp release, update the shared changelog shown on the About page. Its
entries must describe that release's public changes compared with the previous public release and
must use the same product-significance ordering as the release notes.

Update the changelog regression test to assert the new release's important entries, and verify the
changelog renders in the About UI. GitHub release notes and the Announcements discussion do not
replace this requirement. An outdated or unverified in-app changelog blocks the release tag.

## Repository and Issue Workflow

Follow [`docs/development-workflow.md`](docs/development-workflow.md). GitHub is Naviamp's canonical
home for source, issues, pull requests, checks, tags, and releases. Forgejo is a manually maintained
secondary mirror and must not be used to merge changes or publish releases.

Use a GitHub issue and a dedicated short-lived branch for each feature, bug fix, or meaningful
update. Merge completed work into `main` through a linked pull request, use milestones to select
release scope, and cut a short-lived release branch from an accepted `main` commit for final
stabilization.

## GitHub CLI Authentication on Windows

This Windows workspace stores GitHub CLI credentials in Windows Credential Manager. The workspace
sandbox cannot read that keyring, so a sandboxed `gh auth status` or authenticated `gh` command may
falsely report that the token is invalid.

Run every authenticated `gh` command in the Windows credential context outside the workspace
sandbox. Before initiating any login flow, verify the existing credential there with an
authenticated API request such as `gh api user`. Never replace or refresh the credential solely
because a sandboxed authentication check failed.

## Core Is the Product

These are hard architecture requirements, not preferences. Naviamp is one shared application with
thin Android, Desktop, and iOS hosts. A feature is not complete if its behavior must be implemented
again for another platform.

All agents must begin every implementation in common code. Do not prototype, repair, or temporarily
wire product behavior in a platform module with the intention of extracting it later. If a common
owner does not exist yet, create that owner first.

Common ownership includes:

- Product behavior, state, UI, actions, menus, navigation, validation, orchestration, scheduling,
  retry policy, lifecycle policy, feature capability decisions, and user-facing status policy.
- Provider protocol behavior, authentication/session renewal, response interpretation, provider
  model mapping, and provider-specific persistence mapping. Put these in the provider's
  `commonMain` code when they are not provider-neutral; do not put them in a host.
- Persistence schemas, repository behavior, serialization, migrations, cache policy, and storage
  coordination that can use shared SQLDelight or opaque storage contracts.
- Complete shared interfaces and default behavior needed for a new thin host to connect, browse,
  navigate, and render the application without reconstructing features.

## Mandatory Placement Test

Before creating **or modifying** production code in any Android, Desktop, or iOS source set, answer
this question:

> Which concrete operating-system API, native library ABI, or host lifecycle rule makes this code
> impossible to compile and behave correctly in common Kotlin?

If there is no specific answer, the code MUST live in `core`, shared storage, or provider
`commonMain`. “The current caller is Desktop,” “Android already has a controller,” “this is faster
to test here,” and “the shared interface does not exist yet” are not valid answers.

When uncertain, stop and place the behavior in common code. Ask the user only when two genuinely
different product behaviors are required; never assume a platform difference.

## What May Remain Platform-Specific


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goosepod/naviamp](https://github.com/goosepod/naviamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
