---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Setup

Install git hooks before starting work:
```bash
scripts/install-hooks.sh
```

This installs the pre-push hook that enforces build verification.

---

## Component Versioning Policy

The independently releasable components tracked in `VERSION_MANIFEST.md` use
[Semantic Versioning](https://semver.org/) and component-prefixed tags. Prerelease
tags may append a SemVer prerelease identifier, for example
`gnostr-v0.1.0-preview`; their authoritative version sources retain the base
version (`0.1.0`), while the manifest records the full prerelease version. Any change
to a tracked component must assess its version impact before merge. A breaking
change must not land without the corresponding version-source update.

Public surfaces include installed headers and ABI, exported functions/types,
CMake or pkg-config contracts, plugin APIs, supported CLI flags or
machine-readable output, and documented configuration, storage, or wire formats.
For each affected component, apply the highest bump required by the change:

- **MAJOR**: For a stable component (`1.0.0` or later), make any
  backward-incompatible public-surface change. Examples include removing or
  renaming an API, changing a function signature or public struct layout,
  changing documented behavior in a way that breaks callers, or requiring an
  incompatible config/storage/wire migration.
- **MINOR**: Add backward-compatible public functionality, add an optional
  protocol/format capability, or deprecate (without removing) a public API. For
  a `0.x` component, which is explicitly in initial development, use a MINOR
  bump for a breaking public-surface change and call out the incompatibility in
  the release notes.
- **PATCH**: Make a backward-compatible bug or security fix, performance
  improvement, packaging/build correction, or other shipped change that adds no
  incompatible behavior or public API.
- **No bump**: Purely internal, test-only, or documentation-only changes that do
  not alter a shipped artifact. State that assessment in the bead or review
  summary.

Reset lower-order fields after a bump and apply one bump per component, not one
repository-wide bump. If a change affects multiple components, evaluate and bump
each independently.

### Updating versions and the manifest

Until release automation is implemented, version maintenance is deliberately
manual because several components have duplicate CMake/Meson version sources and
`libnostr` and `libgo` do not yet have authoritative version declarations.

1. Identify every affected component and the required bump using the criteria
   above.
2. Update every authoritative source listed for that component in
   `VERSION_MANIFEST.md`; duplicated CMake and Meson values must remain equal.
3. In the same commit, update the manifest's **Declared version**. Do not change
   **Latest release** until that exact version has a matching published
   `<component>-v<MAJOR>.<MINOR>.<PATCH>[-<PRERELEASE>]` tag. For a prerelease,
   keep **Declared version** at the base `MAJOR.MINOR.PATCH` and record the full
   prerelease version under **Latest release**.
4. During a release, verify the declared version against all listed sources and
   existing tags, create/push the component tag, then update **Latest release**
   and **Release tag** in the manifest.
5. Mention all version decisions (including “no bump”) in the bead and peer
   review request.

The planned release-tagging script must validate source/manifest agreement and
update the release columns automatically. Until it lands, the checklist above
is the required process.

---

## Pre-Push Requirements (MANDATORY)

Before pushing ANY commits, you MUST complete these checks:

### 1. Build Verification

The pre-push hook enforces this automatically, but you should verify locally:

```bash
# Clean build
rm -rf _build && cmake -B _build && cmake --build _build
```

**If build fails, DO NOT PUSH. Fix the issue first.**

### 2. Unit Tests

If touching code in `apps/`, `lib*/`, or `nips/`:

```bash
ctest --test-dir _build --output-on-failure
```

All tests must pass before push.

### 3. Peer Review (REQUIRED)

Before pushing, get another agent to review your changes:

1. **Create diff summary:**
   ```bash
   git diff HEAD~1 --stat
   git log -1 --pretty=format:"%s%n%b"
   ```

2. **Send for review:**
   ```bash
   gt mail send nostrc/<other-agent> -s "Review: <bead-id>" \
     -m "<description of changes and diff summary>" --type review
   ```

3. **Wait for approval** - Do NOT push until you receive APPROVED

4. **Push only after approval:**
   ```bash
   git push
   ```

### Reviewer Responsibilities

When you receive a review request:

1. Read the diff and description
2. Check for:
   - Code correctness
   - Obvious bugs or regressions
   - Pattern consistency with codebase
   - Missing error handling
3. Reply with either:
   - `APPROVED` - Author can push

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chebizarro/nostrc](https://github.com/chebizarro/nostrc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
