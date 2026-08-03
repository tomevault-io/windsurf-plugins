---
trigger: always_on
description: Claude Code reads this file at the repo root before doing work here. Follow it.
---

# CLAUDE.md — instructions Claude reads first

Claude Code reads this file at the repo root before doing work here. Follow it.

## First: are you the maintainer or a contributor?

Check the current git identity before committing or releasing:

```
git config user.name && git config user.email
```

- **Maintainer mode** — ONLY when the identity is exactly
  `Lauri Ojansivu <x@xet7.org>` (name `Lauri Ojansivu`, email `x@xet7.org`). Then, and
  only then: commit **directly to the current branch** as `Lauri Ojansivu <x@xet7.org>`
  with no AI trailer and no pull request, and the **publishing / release steps** below
  are available. (Per the standing rule you still **commit only; do not push** unless
  explicitly asked.)
- **Contributor mode** — any other git identity. Then: do **not** commit directly to
  the branch and do **not** run any release/publishing step. Make changes on a branch
  and open a **pull request** for the maintainer to review. The "commit as Lauri
  Ojansivu", "commit directly", and all release instructions below are **maintainer-only
  and do not apply to you**.

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
   Thanks to (GitHub nickname of the issue creator) and xet7 !

   Fixes #1234,
   ```
6. **If the bug is already fixed**, still add a commit that CLOSES the issue (a commit
   whose message ends `Fixes #1234,`), noting where/when it was fixed.

Fix from source and test — do not guess. If the environment cannot run the relevant
test (e.g. Docker is unavailable in this sandbox), reproduce as closely as possible
from source and say clearly what was and was not verified.

## Translations (Transifex ↔ direct LLM fill, no external service)

WeKan translations live in `imports/i18n/data/<lang>.i18n.json` (flat
`key -> string`, 2-space indent, key order matches `en.i18n.json`). Transifex holds
the human translations. **The policy is: never overwrite a human translation with a
filled (or English) one, but always take the newest translations from Transifex.**

- Pull with `releases/translations/pull-translations.sh`. It runs `tx pull -a -f`
  (which fills every string that is UNtranslated on Transifex with the English source)
  and then a **per-key merge** (`releases/translations/merge-translations.mjs`) that,
  for every language file and every key:
  - Transifex has a real translation (pulled value differs from English) → **keep it**
    (the newest human translation always wins);
  - the pull reverted it to English but a human translation is committed in git →
    **restore the committed translation** (a pull never reverts a human translation,
    even in files that also received real new Transifex translations);
  - **no translation anywhere** (untranslated on Transifex AND never committed) → leave
    the English source as a placeholder. **This is the only case a non-human value is
    used.** A separate fill step may fill *only* these English placeholders, so a filled
    string can never overwrite a human translation.
- Restored languages are pushed back to Transifex so they stop reverting.

### Filling the remaining untranslated strings — directly, no translation service

The strings still equal to the English source after the merge are the ones untranslated
**everywhere** (Transifex + git). Translate these **directly** — the maintainer or the
assistant (an LLM) writes the translation itself, using that language's **existing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wekan/wekan](https://github.com/wekan/wekan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
