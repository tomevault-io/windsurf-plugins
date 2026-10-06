---
trigger: always_on
description: This page is the reference for `entire graph init-agents`: what it writes, how
---

# Agent activation

This page is the reference for `entire graph init-agents`: what it writes, how
reruns behave, how Claude inheritance is handled, how to verify that a coding
agent loaded the guide, and how to recover from invalid instruction files. The
behavior described here matches the 0.4.0 release target. Run
`entire graph init-agents --help` for the flags in your installed version.

See [the coordination contract](agent-coordination.md) for generation-time mode
selection, read-only previews, legacy-guide migration, and configuration removal.

## What `init-agents` writes

Activation is per repository. From the repository root:

```sh
entire graph init-agents --repo .
```

The command manages three repository paths:

- `.entire/agent-guide.md`: the complete operating guide for coding agents.
  It is regenerated on every successful run, so manual edits do not survive.
  Its content is identical to `entire graph agent-guide` output.
- `AGENTS.md`: the canonical cross-agent entry point. The command creates the
  file if absent, appends one managed block if no block exists, or replaces the
  existing managed block while preserving other content.
- `CLAUDE.md`: a Claude Code entry point whose managed block is selected from
  the repository's existing instruction-file topology.

The direct managed block contains a pointer to the generated guide. Its
identifying lines are:

```markdown
<!-- entire-agent:begin -->
...
@.entire/agent-guide.md
<!-- entire-agent:end -->
```

A client that resolves `@` imports loads the guide into context. A client that
treats the block as plain text still receives an instruction to read
`.entire/agent-guide.md` before exploring code.

## Claude inheritance

`AGENTS.md` always receives the direct guide pointer. The `CLAUDE.md` block
depends on how that file already reaches `AGENTS.md`:

- If a distinct `CLAUDE.md` contains a live standalone import whose path
  resolves to the root `AGENTS.md`, its managed block contains only this notice:

  ```markdown
  <!-- entire-agent:begin -->
  <!-- Entire agent instructions are inherited through AGENTS.md. -->
  <!-- entire-agent:end -->
  ```

  The user's `AGENTS.md` import remains in place, and the guide is not imported
  directly a second time.
- If that import is absent or cannot be identified safely, `CLAUDE.md` receives
  the direct guide pointer. Removing or adding a live `AGENTS.md` import and
  rerunning `init-agents` switches the managed block accordingly.
- If `AGENTS.md` and `CLAUDE.md` resolve to the same regular file through a
  symlink or hard link, the shared file receives the direct block once. The
  link topology is preserved.

Import detection recognizes standalone relative or absolute paths that resolve
to the root `AGENTS.md`. Mentions inside inline code, fenced or indented code,
HTML comments, or the managed block do not count. Ambiguous Markdown keeps the
direct pointer rather than assuming inheritance. `init-agents` does not create
or remove the user-owned `AGENTS.md` import itself.

## Preflight and rerun behavior

Before its first write, `init-agents` inspects both instruction paths, reads
each distinct file, validates the marker layout, and renders all managed
content from that validated snapshot.

Each path must be missing or resolve to a regular file. Symlinks to regular
files are supported, with the target written either relatively or as an
absolute path, as long as it stays inside the project root; a symlink that
resolves outside is refused and nothing is installed. Staying inside the project
root is necessary but not sufficient: a symlink that lands inside a git
directory — `.git` at any depth, including a nested checkout's and a linked
worktree's `.git` pointer — is refused as well. `.git` is inside the project root
but is not project content, and no instruction file belongs there; writing a
managed block into `config` or a hook would corrupt the repository rather than
configure an agent. The git directory is recognised by its structure rather than
by its name, so a repository whose administrative directory is not called `.git`
— `git init --separate-git-dir=admin`, or a checkout driven by `GIT_DIR` — is
covered by the same refusal.

The landing must also be an agent-instruction file. An alias exists so that
`AGENTS.md` and `CLAUDE.md` can share one instruction file, so a target that is
markdown by extension, or a rules file such as `.cursorrules`, is written; a
target that is some other existing file — a `Makefile`, `.envrc`, or
`.github/workflows/ci.yml` — is refused rather than having a managed block
appended to it. A target that does not exist yet is still created, which is what
the dangling-alias case below relies on, and a target this command wrote on an
earlier run stays writable whatever it is named. Hard links are supported only
between `AGENTS.md` and `CLAUDE.md`, which may share one inode and are updated
once. The generated `.entire/agent-guide.md` guide must remain a distinct file.
A managed target whose inode carries any other name is refused before anything
is written, because an inode's other names cannot be read back from the file —
`ln .git/config CLAUDE.md` resolves to `CLAUDE.md`, spells no `.git` component and looks like an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entireio/entire-graph](https://github.com/entireio/entire-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
