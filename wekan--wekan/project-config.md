---
trigger: always_on
description: Codex reads this file at the repo root before doing work here. Follow it.
---

# AGENTS.md — instructions Codex reads first

Codex reads this file at the repo root before doing work here. Follow it.

## First: who maintains this, and who is committing?

**WeKan, the `wekan/` repositories cloned under `.tools/`, and
[Secretchronicles/TSC](https://github.com/Secretchronicles/TSC) are all maintained by
Lauri Ojansivu (xet7) `<x@xet7.org>`** — [wekan/wekan](https://github.com/wekan/wekan),
[wekan/FerretDB](https://github.com/wekan/FerretDB),
[wekan/node-patches](https://github.com/wekan/node-patches),
[wekan/mongo-tools-patches](https://github.com/wekan/mongo-tools-patches) and TSC,
which is under the **Secretchronicles** organisation rather than **wekan** and is his
all the same — his GitHub profile, [xet7](https://github.com/xet7), says exactly that:
*"WeKan and TSC maintainer"*. Work done on the maintainer's behalf is committed as
**`Lauri Ojansivu <x@xet7.org>`** — that author, in every one of those repositories,
every time. Two rules follow from it and neither has an exception:

- **Never attribute a commit to an AI.** No `Co-Authored-By:` trailer, no "Generated
  with", no assistant or model name — not in the commit message, not in a pull-request
  body, not in the CHANGELOG. [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) is where this
  comes from: *"For pull requests, mention only those participants that are
  **human**."* A `Thanks to ... and xet7 !` line credits people — the issue reporter
  and xet7 — never a tool.
- **If the git identity is missing or wrong in one of these checkouts, set it; do not
  commit under something else.** The `.tools/` clones are made by `build.sh` inside
  this checkout and can come up with no `user.name`/`user.email` of their own, which
  would silently author a commit as whatever the machine's default is:

  ```
  git -C .tools/<repo> config user.name  'Lauri Ojansivu'
  git -C .tools/<repo> config user.email 'x@xet7.org'
  ```

Check the current git identity before committing or releasing:

```
git config user.name && git config user.email
```

- **Maintainer mode** — the identity is `Lauri Ojansivu <x@xet7.org>` (name
  `Lauri Ojansivu`, email `x@xet7.org`), or it is unset in a checkout of one of the
  repositories above, which means it is to be SET to that as above rather than worked
  around. Then: commit **directly to the current branch** as
  `Lauri Ojansivu <x@xet7.org>` with no AI trailer and no pull request, and the
  **publishing / release steps** below are available.
- **Contributor mode** — the identity is somebody ELSE, in a fork or a clone of your
  own. Then: do **not** commit directly to the branch and do **not** run any
  release/publishing step. Make changes on a branch and open a **pull request** for
  the maintainer to review, and keep that pull request free of AI attribution too.
  The "commit as Lauri Ojansivu", "commit directly", and all release instructions
  below are **maintainer-only and do not apply to you**.

Maintainer mode covers TSC as well: commit directly to its `devel` branch, no pull
request, same author and no AI attribution. What it does NOT bring along is WeKan's
house style — TSC keeps its own **GNU ChangeLog** `CHANGELOG` and its own release
process, because a project is read on its own terms (see the CHANGELOG section below).
The one repository under `.tools/` that is somebody else's is `sandstorm-io/sandstorm`,
cloned for reference only.

Everything below marked as maintainer-specific (committing directly, the exact commit
author, and the entire "Making a release" / publishing flow) applies only in maintainer
mode. The rest (translation policy, CHANGELOG structure, tests, validating from code) is
good practice for everyone.

## Process: "Fix open issues"

When asked to fix an open issue (one issue at a time):

1. **Read the whole issue** — the description AND all comments (e.g.
   `gh issue view <n> --comments`, or the GitHub API `.../issues/<n>/comments`).
2. **If it is a bug, first check whether it is already fixed** in the current WeKan
   AND FerretDB source code (read the actual code, not just changelogs). It may have
   been fixed already.
3. **If it is NOT fixed yet, fix it**, using the newest documentation and issues of
   the dependencies involved, checking where the newest MAINTAINED dependency lives and
   reading its source and issues:
   - Meteor 3 docs — https://docs.meteor.com/
   - Meteor 3 source + issues — https://github.com/meteor/meteor/issues
   - other Meteor software docs/code (e.g. Rocket.Chat) for how they solved it
   - Meteor forums — https://forums.meteor.com
   - AtmosphereJS — https://atmospherejs.com/ ; Packosphere — https://packosphere.com/
   - Meteor Community Packages — https://github.com/meteor-community-packages
   - Meteor-Files — https://github.com/veliovgroup/Meteor-Files
   - npm — https://www.npmjs.com
4. **After fixing, add tests** — unit tests, negative tests, and UI tests where
   appropriate — and **run the new tests to verify they pass** (see Tests / the sandbox
   build+test instructions in `docs/Security/Sandboxes/vscode/README.md`).
5. **Commit** (maintainer only — when the current git user is `Lauri Ojansivu
   <x@xet7.org>`), with no "Co-Authored" / AI trailer, a message body ending:

   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wekan/wekan](https://github.com/wekan/wekan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
