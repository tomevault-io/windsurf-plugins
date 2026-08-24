---
trigger: always_on
description: Normative reference for commit messages, versioning, branch and tag names, and
---

# Conventions

Normative reference for commit messages, versioning, branch and tag names, and
file naming.

See [CONTRIBUTING.md](../CONTRIBUTING.md) for the contribution workflow and
[docs/hacking.md](hacking.md) for the development environment.

## 1. Commit messages

Commit subjects follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).

### 1.1 Format

```
<type>[(<scope>)][!]: <description>

[optional body]

[optional footers]
```

Rules:

* Limit the subject line to 72 characters.
* Write the description in the imperative mood: "add", not "adds" or "added".
* Start the description in lowercase. Do not end it with a period.
* Include a `Signed-off-by:` footer in every commit, as required by
  [CONTRIBUTING.md](../CONTRIBUTING.md). Conventional Commits governs the
  subject line; the Developer Certificate of Origin adds a trailer. Both apply.
* Write the pull request title as a valid Conventional Commit.

The pull request title requirement follows from the merge strategy. Pull
requests are squash-merged: pull request #3489, titled `fix: Avoid NPE when
attempting to edit a non-existing secret`, landed on `master` as the
single-parent commit `eb1368e4` with the subject `fix: Avoid NPE when
attempting to edit a non-existing secret (#3489)`. The pull request title, not
the individual commit subjects, is the string that reaches `CHANGELOG.md`.

### 1.2 Types

This list is exhaustive. Reject any other type.

| Type | Meaning | SemVer impact | Changelog section |
|---|---|---|---|
| `feat` | New user-visible capability | MINOR | Added |
| `fix` | Bug fix | PATCH | Fixed |
| `security` | Security fix or hardening | PATCH | Security |
| `perf` | Performance improvement, no behaviour change | PATCH | Changed |
| `refactor` | Internal restructuring, no behaviour change | PATCH | Changed |
| `revert` | Reverts a previous commit | PATCH | Changed |
| `deps` | Dependency version change | PATCH | Changed |
| `docs` | Documentation only | none | omitted |
| `test` | Tests only | none | omitted |
| `build` | Build system: Makefile, goreleaser, Dockerfile | none | omitted |
| `ci` | GitHub Actions, linter configuration, workflows | none | omitted |
| `chore` | Housekeeping that fits nothing above | none | omitted |

Two entries extend the set suggested by the specification:

* `security` populates the `Security` section of the changelog, which
  Keep a Changelog defines as a first-class category. Five commits since
  2025-01-01 already use it.
* `deps` is the preferred type for hand-written dependency commits.
  Accept `chore(deps)` as well: Dependabot emits it, and 147 of the 344
  commit subjects since 2025-01-01 carry the `chore` type, nearly all of
  them dependency bumps.

The following appear as commit types in the history and are not types. Use them
as scopes: `otp` (2 commits since 2025-01-01), `age`, `bug`, `fscopy`, and
`openbsd` (1 each).

### 1.3 Scopes

The scope is optional. Prefer to supply one. Do not invent scopes; omit the
scope when none fits. Omit the scope for a change spanning several areas rather
than listing more than one.

**Commands** — the 41 top-level subcommands: 39 registered by
`(*Action).GetCommands` in `internal/action/commands.go`, plus `pwgen` from
`internal/action/pwgen` and `completion`, both added by `getCommands` in
`main.go`:

```
alias audit cat clone completion config convert copy create delete doctor edit
env find fsck fscopy fsmove generate grep history init insert link list merge
mounts move otp process pwgen rcs recipients reorg setup show sum sync
templates unclip update version
```

**Backends:** `age`, `gpg`, `plain`, `cryptfs`, `fossilfs`, `fs`, `gitfs`, `jjfs`

**Subsystems:** `action`, `audit`, `backend`, `cache`, `completion`, `config`,
`create`, `cui`, `editor`, `hashsum`, `hook`, `notify`, `out`, `queue`,
`recipients`, `reminder`, `store`, `store/leaf`, `store/root`, `tpl`, `tree`,
`updater`

**Public API:** `pkg/gopass`, `api`, `secrets`, `appdir`, `clipboard`,
`ctxutil`, `debug`, `fsutil`, `otp`, `passkey`, `pinentry`, `protect`, `pwgen`,
`qrcon`, `set`, `tempfile`, `termio`

**Meta:** `deps`, `release`, `changelog`, `docs`, `adr`, `ci`, `build`

Write `pkg/gopass` out in full as a scope. Public-API changes must be greppable
against the [A-12](adr/A-12-pkg-api-stability.md) stability contract.

### 1.4 Breaking changes

This repository has two independent compatibility surfaces. Mark them
differently. Do not conflate them.

| Surface broken | Marker | Version impact |
|---|---|---|
| gopass CLI: commands, flags, output format, exit codes, config keys, store format | `!` after the type or scope, **and** a `BREAKING CHANGE:` footer | MAJOR |
| Go module `pkg/gopass` only: an exported symbol removed or changed | a `PKG-BREAK:` footer, **no** `!` | none; MINOR at most |
| Both | `!` + `BREAKING CHANGE:` + `PKG-BREAK:` | MAJOR |

Do not mark a module-only break with `!`. No CLI user can observe such a
change, and `!` demands a major release.

Example of a module-only break, per [A-12](adr/A-12-pkg-api-stability.md):

```
refactor(pkg/gopass): drop Store.GetRevision in favour of Store.History

Deprecated since 1.17.0; the two-minor / three-month window has elapsed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gopasspw/gopass](https://github.com/gopasspw/gopass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
