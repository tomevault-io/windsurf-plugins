---
trigger: always_on
description: **This file is auto-loaded into EVERY session for this project.** Its size is a
---

# Clayrune — Claude Code project notes

**This file is auto-loaded into EVERY session for this project.** Its size is a
per-session context tax on every agent, forever — so it holds only what an agent
must know *before* it starts, not the history of how things got this way.

The bar for a section here: **an agent that reads the code carefully still could
not derive it.** Binding constraints, invariants whose violation is silent, and
incidents that already happened. Design narrative, build orders and shipped-work
status belong in `docs/` — see `docs/CLAUDE_MD_ARCHIVE.md` for what was moved
out on 2026-08-06 and why.

## Commit discipline — stay scoped to your own session (added 2026-06-08)

When asked to commit "the work we did," stage **only the files you edited this
session, by explicit path** (`git add <path> …`):

- **Never** `git add -A`, `git add .`, or `git commit -a`. Name the paths.
- **Don't sweep, don't narrate.** The working tree always carries unrelated
  dirty files — other MC-managed projects' data under `data/projects/`,
  backups, `_scratch/`, mobile/store assets. Don't stage them; don't list them
  back. A commit report names only what you committed. Enumerate other dirty
  files **only** if the user asks "what else is uncommitted?"
- Scratch/throwaway artifacts go in **`_scratch/`** (gitignored), not in
  `tools/`, `docs/`, or `data/`.
- Pairs with the standing rule to commit your own completed work without asking
  — but *only your own*.

`.gitignore` enforces the structural half: backups (`*.bak`/`*.broken`),
runtime (`data/mc_child_pids.json`, `data/skills/_proposed/`), `_scratch/`,
`tools/_*` scratch, served build artifacts, and mobile-app assets are all
untracked so they never reach a commit candidate.

## BINDING — `master` is the release channel: keep it pushed (added 2026-07-12)

`/api/system/update` is `git pull --ff-only` on the user's **current branch** —
for everyone who isn't us, that's `master`. So **an unpushed `master` means every
other user is frozen on old code**, silently. There is no other release channel.

On 2026-07-12 `origin/master` was found **138 commits behind local `master`** and
the working branch was 33 commits beyond that. Months of shipped work — the
conversation redesign, Inbox, mobile fixes, safety rails — had reached nobody.
Nothing warned us; "it works on this machine" is exactly what that failure looks
like.

**The rule:** when a feature branch is done and green, land it on `master` and
**push `master`** in the same breath. Do not let the local `master` accumulate
commits that `origin/master` doesn't have. Merging locally is not shipping;
pushing is.

**Check it costs one command** — run it at the end of any session that landed
work, and whenever the user asks whether something "is out there":

```bash
git rev-list --left-right --count origin/master...master   # want: 0  0
```

Left number > 0 = we're behind the remote. **Right number > 0 = users are behind
us, and won't know it.** That second case is the dangerous one.

## BINDING — nothing operator-specific goes in the repo (added 2026-07-12)

This repo is public and its files are consumed by other people's machines *and
other people's agents*. Anything true only of **this** install is user data, not
source. Before committing, ask: "would this be wrong on a stranger's machine?"

The three that actually bit us (all fixed 2026-07-12, commit `3a1fd04`):

- **Rules files are the sharpest edge.** `data/SHARED_RULES.md` is read verbatim
  into the system prompt of **every agent on every project**, and a project's
  `AGENT_RULES.md` does the same for that project. Committing them injected one
  operator's personal working preferences — and their email — into every other
  install's agents. Both are now gitignored. A fresh install starting with no
  rules is the **correct** default; the Rules editor writes them.
- **Gitignoring is not enough if a build bundles the file.** `build-macos.spec`
  packaged `data/SHARED_RULES.md` *"if present"* — and it is still present on
  the builder's disk after being ignored, so it got baked into the shipped
  `.app` anyway. When you untrack something, **also check the build specs and
  installer for it.**
- **Personal identity/paths belong in the environment.** Signing identity →
  `tools/signing.env` (gitignored). Machine paths → derived, or `MC_DIR` /
  `JAVA_HOME` / `ANDROID_HOME` / `CLAYRUNE_MOBILE_REPO`. Recipients → config,
  never a hardcoded default. Private ops tooling that audits *our* accounts
  (`tools/gcp-cost-review/`, its billing-account ID) stays untracked.

Legitimately public and intentionally kept: the `LICENSE` copyright line, and
the Play Store `PRIVACY_POLICY.md` / `LISTING_COPY.md` contact address.

## BINDING — credentials go in the vault, never in a command line (2026-08-01)

`mc/secrets_store.py` is the secrets vault. Full detail: `docs/SECRETS.md`.

**If a task needs a password, token, or API key, reference it by name and let
the server resolve it.** Never type a credential into a command, a config file
under the repo, or a message — anything you type lands in the transcript, and
from there in `MEMORY.md` and possibly a distilled artifact, permanently.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ronle/clayrune](https://github.com/ronle/clayrune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
