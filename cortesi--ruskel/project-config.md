---
trigger: always_on
description: - Never commit code without explicit user confirmation. Just because the
---


# Core Guidelines

- Never commit code without explicit user confirmation. Just because the
  user has consented to one commit doesn't mean they consent to all future commits.
- When removing or changing code, never add comments about what was removed or
  changed. Comments in the code should always reflect what's there in the moment.
- You are an autonomous agent. You make use of all the tools available to you.
  You run instrument code and run tests and smoketests to gather information to
  solve problems. You iterate persistently until your requirements are met.
- You may create temporary files and directories as needed to solve problems,
  but always place them in the `./tmp/` directory.
- ALWAYS lint and fix all warnings before returning to the user.
- As a final step, ALWAYS format code before returning to the user.

# Active Code Maintenance

Every time you touch a piece of code, evaluate whether it can be improved
structurally. Ask questions like:

- Is the documentation for this function clear, concise and acccurate?
- Should the code be broken up into smaller pieces?
- Can the code be generalized or made more flexible?
- Is there related code that should be refactored to share functionality?
- Should the code be moved to a different location in the project?

You should improve the code continuously when opportunities arise, even if the
user hasn't explicitly asked for it. When you do active maintenance, include an
"Active Maintenance" section in your response message.

# Checklists

Whenever you're asked to produce a todo list or a checklist, you will use a
Markdown checklist, with numbered sections and items. Each item should be a
single, coherent change that leaves the system in a consistent state. Try not
to leave a broken system after any step, but certainly after a stage all tests
and smoketests must pass. Always include enough information that you could pick
it up again with zero context. Always wrap at 100 chars.

The checklist is a LIVE DOCUMENT, update it as you go - if you discover new
items during your work or leave items for a later commit, add them to the
checklist. Ensure that any new item you add is a also Markdown checklist item
(i.e. starts with `[ ]`), and has a number in sequence with other items in the
document. You should evaluate next steps continuously, and modify the checklist 
to incorporate what you learn as you work.

You may batch together todo items that you think belong in the same changeset
without prompting me. After every batch, let me review the code before
committing. 

IMMEDIATELY tick off each item in the original checklist file on disk as you
complete them, so we don't lose track of where we are. Don't confuse your own
checklist with the user's checklist - update both your internal checklist and
the checklist on disk.

Example format:

```markdown
# Task description

Here is the context needed to understand the task, and an outline of its broad
aims.

1. Stage One: Frobnitz the flange

Perhaps some explanation and comments go here.

1. [ ] Do a thing!
2. [ ] Do thing two.

3. Stage Two: Retrofit the turbo-enabulator

Perhaps some explanation and comments go here.

1. [ ] Second section thing.
2. [ ] Second section thing 2.
```

# Git Commits 

Never commit until you're asked to do so, or the user has explicitly confirmed
they want to commit (the user will say "commit" or "do a git commit" or some
variant of that). Make git commit messages concise and clear. In the body of
the message, provide a concise summary of what's been done, but leave out
details like the validation process. Commit as the user - don't add model
attribution or co-authorship.

First, review the actual changes that are being committed.

```sh
# 1) Review, then stage explicitly (paths or -A).
git status --porcelain

# If necessary, review changes before staging:
git diff 
```

Formulate your commit message, based on the actual diff and the user's
instructions that lead up to this point. Make sure your message covers all
changed code, not just the user's latest prompt.

Next, stage and commit:

```sh
# Stage changes; use -A to stage all changes, or specify paths.
git add -A  # or: git add <paths>

# Commit via stdin; Conventional Commit subject (≤50). Body optional; blank
# line before body; quoted heredoc prevents interpolation.
git commit --cleanup=strip -F - <<'MSG'
feat(ui): concise example

Body
MSG
```



# Rust Development Guidelines

## General Guidelines

- Use Rust edition 2024.
- Avoid including code examples in documentation comments.
- Always introduce items from the standard library with a `use` declaration at
  the top of the file; do not reference `std` paths directly in the code body.
- Functions should not be nested inside other functions except in extremely
  rare cases where necessary.
- You should amost never add dead_code annotations. If you find yourself doing
  this, default to removing the code instead, unless there's a very good reason.
- Avoid feature gating unless absolutely necessary.
- Avoid configuring tests or any code component with environment variables. Prefer
  using function parameters or configuration structs.


## Crate structure

- Every crate that has methods that return `Result` should have a custom error

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cortesi/ruskel](https://github.com/cortesi/ruskel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
