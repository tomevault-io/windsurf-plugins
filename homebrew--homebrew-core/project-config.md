---
trigger: always_on
description: 1. **Check for existing PRs** for the same formula: [open PRs](https://github.com/Homebrew/homebrew-core/pulls)
---

# Agent Instructions for homebrew-core

## Before Any Changes

1. **Check for existing PRs** for the same formula: [open PRs](https://github.com/Homebrew/homebrew-core/pulls)
2. Run `brew tap homebrew/core` if not already tapped
3. Read the resources listed in References

## Formula Changes

Best way to learn is by example. Check and use existing formulae as reference of how to do specific things. See the resources at the end for links to documentation and examples (mandatory). Many failures or problems may already have been encountered by other formulae.

### New Formulae

- Before anything check if the software qualifies for inclusion by reading [Acceptable Formulae](https://docs.brew.sh/Acceptable-Formulae). Especially make the user aware of the notability requirements and alternative options until notable.
- Follow the [Formula Cookbook](https://docs.brew.sh/Formula-Cookbook) and use existing formulae as examples
- Create by running `brew create <url>` and then edit the generated formula
- Install from source with `HOMEBREW_NO_INSTALL_FROM_API=1 brew install --build-from-source <formula>` and optionally with `-vd` to verify it builds correctly
- Run tests with `HOMEBREW_NO_INSTALL_FROM_API=1 brew test <formula>`; Audit with `HOMEBREW_NO_INSTALL_FROM_API=1 brew audit --new --strict <formula>` after install, before opening PR for new formulae, and fix style and audit with `brew lgtm <formula>` (or `brew style <formula> --fix` to only check style issues)

### Dependencies

- Add new dependencies only when actually required for build or runtime; prefer the use of formulae as dependecies rather than using resources and try to avoid downloads inside install block.
- Avoid externally downloaded binaries, instead use `depends_on "formula"`, or resource blocks if you require external resource

### Required Elements

#### Test block:

- Test MUST verify actual functionality:
  - execute the installed binary or library
  - include at least one assertion beyond `--version` or `--help` or similar
  - validate the build produced a working result,
  - For libraries: compile and link sample code
- `testpath` returns the working directory for tests in test block
- Test should not cover every possible edge case nor be so minimal that it doesn't actually verify the software.

#### Service block:

If the software can run as a daemon, include a `service do` block:

```ruby
service do
  run [opt_bin/"foo", "start"]
  keep_alive true
end
```

#### Livecheck

Prefer default behavior. Only add a `livecheck` block if automatic detection fails.

#### Head support

Include when the project has a development branch:

```ruby
head "https://github.com/org/repo.git", branch: "main"
```

- Git repositories MUST specify `branch:`.
- If HEAD is added Formula MUST build or must be made to build from head, or HEAD support should be removed.

### Version Updates

Preferred method for version bumps: (see also `brew bump-formula-pr --help` for flags to e.g. update git tags and more)

```sh
brew bump-formula-pr --strict <formula> --version=<version>
```

This handles URL/checksum updates, commit message, and opens the PR automatically.

#### Manual Version Updates

If manual update is needed manually update `url` and `sha256` (or `tag` and `revision`)

#### When to Add a Revision

Run `brew bump-revision [--write-only] <formula>` when:

- Fix requires existing bottles to be rebuilt
- Dependencies changed in a way that affects the built package
- The installed binary/library behavior changes

Do NOT add revision for cosmetic changes (comments, style, livecheck fixes).

## Required Validation (All PR Types)

All checks MUST pass locally before opening a PR (add `--debug` and/or `--verbose` to command if more output is needed):

```sh
# Build from source (required)
HOMEBREW_NO_INSTALL_FROM_API=1 brew install --build-from-source <formula>

# Run tests
brew test <formula>

# Audit (existing formula) add --new for new formulae
brew audit --strict [--new] <formula>
```

## CI Failures

- Fetch complete build log in "Checks" (via gh cli) and download logs from ci artifacts
- Reproduce failures locally (if you have not already before opening the PR); For Linux failures, use the [Homebrew Docker container](CONTRIBUTING.md#homebrew-docker-container)
- Code patches added for fixing/patching source must be submitted as pr or opened as an issue upstream if not already done, and linked in the PR description and as a comment in the formula if applicable. So first try without patching.
- If stuck, comment describing what you've tried

## PR Template Checklist

You MUST verify all items before submitting:

- [ ] Followed [CONTRIBUTING.md](CONTRIBUTING.md)
- [ ] Commits follow [commit style guide](https://docs.brew.sh/Formula-Cookbook#commit)
- [ ] No existing [open PRs](https://github.com/Homebrew/homebrew-core/pulls) for same change
- [ ] Built locally with `HOMEBREW_NO_INSTALL_FROM_API=1 brew install --build-from-source`
- [ ] Tests pass with `brew test`
- [ ] Audit passes with `brew audit --strict` (or `--new` for new formulae)

## Commit Message Format

- Version update: `foo 1.2.3`
- New formula: `foo 1.2.3 (new formula)`
- Fix/change: `foo: fix <description>` or `foo: <description>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
