---
trigger: always_on
description: A monorepo of pure-Swift, cross-platform reimplementations of standard
---

# Agent instructions — SwiftPorts

A monorepo of pure-Swift, cross-platform reimplementations of standard
CLI tools and SDK clients. They share one `Package.swift`, one git
history, and one test runner.

## Today's targets

| Library        | Binary | What it ports                                     |
|----------------|--------|---------------------------------------------------|
| `ForgeKit`     | —      | Host-agnostic CLI plumbing: ANSI/TTY, Git client (Process + No-op), Secret store (Keychain + InMemory). Used by `GitHub` and the upcoming `GitLab`. |
| `ZipKit`       | —      | Pure-Swift PKZIP archive operations on top of `weichsel/ZIPFoundation`. Shared by Zip / Unzip / GitHub. |
| `ZipCommand`   | `zip`  | Info-ZIP `zip(1)` — create archives. |
| `UnzipCommand` | `unzip`| Info-ZIP `unzip(1)` — extract / list / test / pipe. |
| `GitHub`       | —      | GitHub SDK: API client, Codable models, OAuth device flow, GraphQL. No ArgumentParser dependency. |
| `GhCommand`    | `gh`   | The `gh` subcommand tree — built on top of `GitHub` + `ForgeKit`. SwiftBash extends `GhCommand` to register the whole tree as a Bash builtin. |
| `GitLab`       | —      | GitLab SDK: REST API client (`X-Next-Page` pagination, Bearer auth, `gitlab.com` and self-hosted instances), Codable models, `RepositoryReference` with nested-subgroup support. No ArgumentParser dependency. |
| `GlabCommand`  | `glab` | The `glab` subcommand tree — built on top of `GitLab` + `ForgeKit`. Today: `issue list / view / create / close / reopen / note / subscribe / unsubscribe / delete / board`, `mr list / view / create / update / close / reopen / merge / approve / unapprove / note / subscribe / unsubscribe / checkout / diff / delete`, `ci list / view / trace / status / retry / cancel / run / lint`, `repo {view,list,create,clone,fork,archive,unarchive,edit,delete}`, `release {list,view,create,delete,download}`, `tag {list,create,delete}`, `variable {list,set,unset}`, `label {list,create,delete}`, `api`, `auth {status,login,logout,token}`. Uses libgit2 via `SwiftGit.GitClient` (no shell-out). |
| `SwiftGit`     | —      | In-process `GitClient` impl backed by libgit2 1.9.x (vendored from `ibrahimcetin/libgit2` SwiftPM package). Drop-in replacement for `ForgeKit`'s `ProcessGitClient` — no system `git` binary required. HTTPS auth via `CredentialProvider` callback. Named `SwiftGit` (not `Git`) so its build artifacts don't case-fold-collide with the lowercase `git` exec on macOS. |
| `GitCommand`   | `git`  | The `git` subcommand tree — `init {--bare,-b <branch>} / clone / fetch / pull {--rebase} / checkout {-b/-B/--/<ref> --} / switch {-c/-C} / restore {--staged, --source} / push / add / reset {--soft,--mixed,--hard,-- <paths>} / status {-s,--porcelain,-b, ahead/behind} / commit / merge {--ff,--no-ff,--ff-only} / rebase {<upstream>,--continue,--skip,--abort,--onto} / cherry-pick {<commit>,--continue,--skip,--abort} / diff / log {--oneline,--format,--stat,-p,-<n>,<a>..<b>,-- <paths>} / show / blame / apply {--cached, --index} / reflog / describe {--tags,--dirty,--abbrev <n>} / ls-tree {-r, --name-only} / cat-file {-t,-s,-e,-p} / rev-parse {--short,--abbrev-ref,--git-dir,--show-toplevel,--is-inside-work-tree} / ls-files / mv / rm {--cached} / clean {-f, -n} / config {--get,--set,--list,--unset,--global,--system,--local} / stash {push,list,apply,pop,drop,clear,show,branch} / tag {-a -m, -d, -l, -n, -f} / remote {-v, add, get-url, set-url, remove, rename} / branch {-d, -D, -m, -M, --show-current} / version`. Output and exit-code semantics mirror real git for every supported case. SwiftBash can register `GitCommand` as the `git` builtin to shadow system git. See [Docs/SwiftGit.md](Docs/SwiftGit.md) for the full module surface. |

## Build, test, run

```bash
swift build                              # builds everything
swift test                               # all targets, all tests (156 today)
swift run gh ...                         # GitHub CLI
swift run glab ...                       # GitLab CLI
swift run git ...                        # libgit2-backed git CLI
swift run zip ...                        # zip(1)
swift run unzip ...                      # unzip(1)
```

`swift build -c release` produces optimized binaries under
`.build/release/`.

## CI

GitHub Actions workflow at `.github/workflows/swift.yml` runs on push
to `main` and PRs. Matrix:

- **macOS** — `macos-15` runner, `swift build && swift test`
- **iOS Simulator** — same runner, `xcodebuild` against the auto-generated
  `SwiftPorts-Package` umbrella scheme
- **Linux** — `swift:6.0-jammy` Docker image
- **Windows** — `windows-latest` + `SwiftyLab/setup-swift`
  (`continue-on-error` until libgit2 builds cleanly there)
- **Android** — `skiptools/swift-android-action`
  (`continue-on-error`, same caveat)

The Windows + Android jobs are stretch goals — `ibrahimcetin/libgit2`
1.9.x doesn't advertise official support and the C build path needs
platform-specific defines we haven't tuned yet.

## Layout — umbrella convention

A port is either:

- **Pure library** — flat folder `Sources/<Name>/`, one library target.
- **Library + binaries** — umbrella folder `Sources/<Umbrella>/` containing:
  - `Lib/` — the SDK library target, named `<Umbrella>` (e.g. `ZipKit`, `GitHub`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cocoanetics/SwiftPorts](https://github.com/Cocoanetics/SwiftPorts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
