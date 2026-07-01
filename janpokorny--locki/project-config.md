---
trigger: always_on
description: You are running inside a Locki sandbox -- an Incus LXC container running in a Lima VM. This environment is designed to give you free reign -- you are running as `root` -- while preventing accidental damage to files on the host machine.
---

# Sandbox environment

You are running inside a Locki sandbox -- an Incus LXC container running in a Lima VM. This environment is designed to give you free reign -- you are running as `root` -- while preventing accidental damage to files on the host machine.

You are operating on a separated worktree folder of a git repo lying outside of the sandbox -- for this reason, `.git` is just a file pointer and you can't access the actual `.git` folder. Git operations are only possible using the command bridge, see below.

The `.locki/tmp/` folder, like the rest of the worktree, is shared with the host -- the user can see its contents. Put screenshots (e.g. from agent-browser), scratch files, debug dumps, and other artifacts there.

The sandbox may also contain **included worktrees** from other repositories under `.locki/include/<repo-name>/`. Each include is a full git worktree of a separate repo; the command bridge rules apply inside each include the same way as in the main worktree (branch/stash ownership is scoped by the sandbox id, so commands work identically). `cd` into the include folder to operate on that repo. If the user asks you to work on multiple repos at once and an include is not yet present, tell the user to run `locki include --repo <path>` (or, from the other repo, `locki include --this -m <this-sandbox>`).

# Command bridge

Some commands execute on the host using a command bridge. This lets you execute a limited safe set of higher-priviledged commands. Run them as usual -- the executables present in sandbox are shims that call out to the bridge. The proxy will reject the command if it does not exactly match an allowed pattern. If user asks you to perform an operation you can't do, you can always prepare commands for them to run on host (worktree path matches 1:1).

## Git

```locki-bridged-command-filter
git -v/--version
git add (--all | <file> ...)
git blame [-L=<range>] <file>
git branch (<name>#locki-<wt-id> [<start-point> | --move | --delete [--force]] | --show-current | [-l/--list] [-a/--all | -r/--remotes] [-v/--verbose] [--contains <ref>] [--merged [<ref>]] [--no-merged [<ref>]] [<pattern>])
git cat-file -t/--type <ref>
git checkout (--ours | --theirs) <file> ...
git check-ignore <file> ...
git cherry-pick (--continue | --abort | --skip)
git cherry-pick [--no-commit] [--gpg-sign] <ref> ...
git commit (-m/--message=<msg> [-s/--signoff] | -C/--reuse-message=<sha> [-s/--signoff] | --amend --no-edit [-s/--signoff]) [--amend [--no-edit]] [--allow-empty] [--gpg-sign]
git config [--get] [--local] <key>
git count-objects [-v/--verbose]
git describe [--tags] [--always] [--abbrev=<n>] [<ref>]
git diff [--staged | --cached] [--name-only] [--stat] [--numstat] [--name-status] [--exit-code] [--diff-filter=<filter>] [-U/--unified=<n>] [-w/--ignore-all-space] [--ignore-space-change] [<ref> [<ref>]] [<file> ...]
git fetch [--prune] [<remote> [<ref> ...]]
git for-each-ref [--format=<fmt>] [--sort=<key>] [<pattern> ...]
git grep [-l/--files-with-matches] <pattern> [<ref>] [<file> ...]
git hash-object <file>
git log [--oneline] [--all] [--graph] [--reverse] [--first-parent] [--stat] [-p/--patch] [--follow] [--no-merges] [--merges] [--decorate] [--left-right] [-S=<string>] [-G=<regex>] [--format=<fmt>] [--pretty=<fmt>] [-n/--max-count=<n>] [--name-only] [--name-status] [--diff-filter=<filter>] [--since=<date>] [--grep=<pattern>] [--author=<author>] [--ancestry-path] [--not] [<ref>] [<file> ...]
git ls-files [--error-unmatch] [--recurse-submodules] [--others] [--exclude-standard] [<path> ...]
git ls-remote [<remote>]
git ls-tree [-r] [-d] [-l/--long] [--name-only] [--name-status] [--abbrev=<n>] <ref> [<file> ...]
git merge (--continue | --abort | --skip)
git merge <ref>
git merge-base [--is-ancestor] <ref> <ref>
git mv <file> <file>
git pull [--rebase] [--ff-only]
git push [-u/--set-upstream] [--force-with-lease] [<remote> [<name>#locki-<wt-id>]]
git rebase (--continue | --abort | --skip)
git rebase <ref>
git reflog [--all]
git remote (-v/--verbose | get-url <remote>)
git reset [--hard] <ref>
git restore [--staged] [--source=<ref>] <file> ...
git rev-list [--count] [--ancestry-path] [<ref>] [<ref>]
git rev-parse [-q/--quiet] [--show-cdup] [--show-toplevel] [--git-dir] [--git-common-dir] [--is-inside-work-tree] [--abbrev-ref] [--symbolic-full-name] [--short] [--verify] [<arg> ...]
git rm [-q/--quiet] [-f/--force] <file> ...
git shortlog [-s/--summary] [-n/--numbered] [<ref>] [<ref>]
git show [<ref>] [--stat] [--oneline] [--name-only] [--name-status] [-s/--no-patch] [-q/--quiet] [--diff-filter=<filter>] [--format=<fmt>] [--pretty=<fmt>] [<file> ...]
git stash [push] [-m/--message=<text>#locki-<wt-id>] [<file> ...]
git stash (pop | drop) [<owned-stash-ref>]
git stash apply [<stash-ref>]
git stash list
git stash show [<stash-ref>]
git status [-s/--short] [-b/--branch] [-u/--untracked-files=<mode>] [--porcelain] [--ignore-submodules] [--ahead-behind] [<file> ...]
git symbolic-ref [-q/--quiet] <ref>
git switch ([--create | --force-create] <name>#locki-<wt-id> [<start-point>] | --detach <ref>)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JanPokorny/locki](https://github.com/JanPokorny/locki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
