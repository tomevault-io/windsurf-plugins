---
trigger: always_on
description: Guidance for coding agents working in this repo. Applies to Claude Code and to
---

# CLAUDE.md — pi-go

Guidance for coding agents working in this repo. Applies to Claude Code and to
pi-go's own agent; where the two differ, both are described.

## Work in a git worktree, not the primary checkout

**Do not make uncommitted edits in `/Users/dimetron/p6s/pi-dev/pi-go` and leave
them there.** The primary checkout has its branch switched frequently, and
`git checkout` discards uncommitted changes in tracked files without warning.
Work has been lost to this. A worktree gives each task its own working
directory and its own branch, so a switch in one cannot destroy another.

Create one before starting any task that edits tracked files:

```bash
git worktree add -b fix/<topic> .worktrees/fix-<topic> HEAD
cd .worktrees/fix-<topic>
```

Remove it when the branch is merged or abandoned:

```bash
git worktree remove .worktrees/fix-<topic>
git worktree list          # verify; prune stale metadata with `git worktree prune`
```

### Where worktrees live

Three conventions coexist. Match the one that fits who is doing the work.

| Creator | Path | Branch | Notes |
|---|---|---|---|
| Human / Claude Code | `<repo>/.worktrees/<branch-with-dashes>` | `fix/…`, `feat/…` | Inside the repo, so it stays within the sandbox; `.worktrees/` is gitignored |
| pi-go agent (`/run`, subagents) | `<repo>/.pi-go/tasks/<pathID>` | `pi-agent-<shortID>`, or the sanitized requested name | Created by `internal/subagent/worktree.go`; `.pi-go/` is gitignored |
| `arbor` tool | `~/.arbor/worktrees/pi-go/<name>` | matches dir name | External tool, listed here only so `git worktree list` output is not surprising |

`.pi-go/` and `.worktrees/` are both gitignored, so agent worktrees never show
up as untracked noise in `git status`.

### What pi-go's agent already does, and why it matters

`WorktreeManager.Create` (`internal/subagent/worktree.go:117`) **stashes
uncommitted changes before `git worktree add` and pops them afterwards**, with
a unique stash message so the pop is deterministic
(`stashMessage`, `popStashByMessage`). It does this because `worktree add` from
HEAD fails on a dirty tree.

The consequence worth knowing: if a pi-go subagent runs while you have
uncommitted work in the primary checkout, your changes take a round trip
through the stash. That is safe, but it is one more reason not to keep
long-lived uncommitted work in the primary checkout.

Agents marked `[worktree]` edit an isolated tree. Their edits do **not** land in
the caller's tree — ask for an explicit patch or file list to apply, or use a
non-worktree editing agent (`internal/tools/subagent.go:127-128`).

## Commits: always sign off and sign

Every commit must carry both a `Signed-off-by` trailer and a cryptographic
signature:

```bash
git commit -s -S -m "..."     # -s = Signed-off-by trailer, -S = sign
```

`-S` is redundant when config is honoured (`commit.gpgsign` and `tag.gpgsign`
are already `true`), but pass it explicitly so a commit fails loudly rather than
landing unsigned when config is missing or overridden.

Signing here is SSH-format, not GPG, through 1Password:

```
gpg.format       = ssh
user.signingkey  = ssh-ed25519 AAAAC3Nza...
gpg.ssh.program  = /Applications/1Password.app/Contents/MacOS/op-ssh-sign
```

### The sandbox breaks signing — commit with it disabled

`op-ssh-sign` needs to reach the 1Password agent, which is not reachable from
inside the sandbox. A commit attempted there either fails or, worse, succeeds
unsigned. **Run `git commit` with the sandbox disabled.**

This is not hypothetical: commits `0714568`, `a8b243b` and `bcb6d26` carry no
signature at all, and only some commits carry a `Signed-off-by`.

**Do not use `%G?` to check.** `gpg.ssh.allowedSignersFile` is not configured, so
signature *verification* cannot run: `git log --show-signature` errors with
`gpg.ssh.allowedSignersFile needs to be configured and exist`, and `%G?` reports
`N` for correctly signed and unsigned commits alike. It is a verification gap,
not a signing failure, but it makes the obvious check useless.

Inspect the raw object instead — a signed commit has a `gpgsig` header:

```bash
git cat-file commit HEAD | grep -q '^gpgsig' && echo signed || echo UNSIGNED
git log --format='%h %(trailers:key=Signed-off-by,valueonly,separator=;) %s' -5
```

Configuring an allowed-signers file would restore `%G?`, and is worth doing:

```bash
echo "dimetron@me.com $(git config user.signingkey)" > ~/.config/git/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers
```

### Never use `--no-verify`

**Do not pass `--no-verify` to `git commit`, ever.** Not to unblock a failing
hook, not "just this once", not with a note in the commit message. There is no
case in this repo where it is the right answer.

`--no-verify` skips *all* hooks, including the signing path — so a bypassed
commit lands unsigned, and because `gpg.ssh.allowedSignersFile` is unset (see
above) nothing will tell you. That is how `0714568`, `a8b243b` and `bcb6d26`
ended up with no signature.

The hook that usually tempts this is `golangci-lint`, which runs against the
**primary checkout** and so can fail on pre-existing issues in files a worktree
branch never touched (currently 10 `SA1019` deprecation errors in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dimetron/pi-go](https://github.com/dimetron/pi-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
