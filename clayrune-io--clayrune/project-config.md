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

## Commit discipline — stay scoped to your own session (2026-06-08)

When asked to commit "the work we did," stage **only the files you edited this
session, by explicit path** (`git add <path> …`):

- **Never** `git add -A`, `git add .`, or `git commit -a`. Name the paths.
- **Don't sweep, don't narrate.** The working tree always carries unrelated
  dirty files (other MC-managed projects' data under `data/projects/`,
  backups, `_scratch/`, mobile/store assets). Don't stage or list them back —
  only enumerate other dirty files if the user asks "what else is uncommitted?"
- Scratch/throwaway artifacts go in **`_scratch/`** (gitignored), not
  `tools/`, `docs/`, or `data/`.
- Pairs with the standing rule to commit your own completed work without
  asking — but *only your own*.

`.gitignore` enforces the structural half: backups (`*.bak`/`*.broken`),
runtime (`data/mc_child_pids.json`, `data/skills/_proposed/`), `_scratch/`,
`tools/_*` scratch, served build artifacts, and mobile-app assets are all
untracked so they never reach a commit candidate.

## BINDING — `master` is the release channel: keep it pushed (2026-07-12)

`/api/system/update` is `git pull --ff-only` on the user's **current branch** —
for everyone who isn't us, that's `master`. No other release channel exists,
so an unpushed `master` freezes every other user on old code, silently.
Incident 2026-07-12: `origin/master` was 138 commits behind local `master`
(local 33 more beyond that) — months of shipped work reached nobody, no
warning.

**The rule:** when a feature branch is done and green, land it on `master`
and **push `master`** in the same breath — merging locally is not shipping.
One-command check, run at the end of any session that landed work:

```bash
git rev-list --left-right --count origin/master...master   # want: 0  0
```

Left > 0 = we're behind the remote. **Right > 0 = users are behind us, and
won't know it** — the dangerous case.

## BINDING — nothing operator-specific goes in the repo (2026-07-12)

This repo is public, consumed by other people's machines *and agents*.
Anything true only of **this** install is user data, not source — ask "would
this be wrong on a stranger's machine?" before committing. Three that bit us
(fixed 2026-07-12, commit `3a1fd04`):

- **Rules files are the sharpest edge.** `data/SHARED_RULES.md` is read
  verbatim into the system prompt of **every agent on every project**; a
  project's `AGENT_RULES.md` does the same for that project. Committing them
  had injected one operator's personal preferences (and email) into every
  other install's agents. Both now gitignored; a fresh install with no rules
  is the correct default — the Rules editor writes them.
- **Gitignoring isn't enough if a build bundles the file.** `build-macos.spec`
  packaged `data/SHARED_RULES.md` *"if present"* — still present on the
  builder's disk after being ignored, so it baked into the shipped `.app`
  anyway. When you untrack something, **also check build specs/installer.**
- **Personal identity/paths belong in the environment.** Signing identity →
  `tools/signing.env` (gitignored). Machine paths → derived, or `MC_DIR` /
  `JAVA_HOME` / `ANDROID_HOME` / `CLAYRUNE_MOBILE_REPO`. Recipients → config,
  never hardcoded. Ops tooling auditing our own accounts
  (`tools/gcp-cost-review/`, billing-account ID) stays untracked.

Legitimately public and kept: the `LICENSE` copyright line, and the Play
Store `PRIVACY_POLICY.md` / `LISTING_COPY.md` contact address.

## BINDING — credentials go in the vault, never in a command line (2026-08-01)

`mc/secrets_store.py` is the secrets vault. Full detail: `docs/SECRETS.md`.

**Reference a credential by name and let the server resolve it.** Never type
one into a command, a repo config file, or a message — anything typed lands
in the transcript, then `MEMORY.md`, possibly a distilled artifact, permanently.

```bash
python tools/with-secret.py --env GH_TOKEN=github.token -- gh api ...
```

Placeholders (`{{secret:name}}`) resolve server-side; `tools/with-secret.py`
injects into the child's environment and scrubs dispensed values from output.

**A login is one entry, not two** — username lives on the same secret as the
password: `{{user:name}}` / `--user VAR=name` (username), `{{secret:name}}` /
`--env VAR=name` (password), `{{totp:name}}` / `--totp VAR=name` (2FA code).
`curl -s localhost:5199/api/secrets` lists what exists (metadata only — names,
usernames, scope; never a value):

```bash
python tools/with-secret.py --user U=reddit --env P=reddit -- python post.py
```

If `{{user:name}}` errors "no username stored", the entry is half-filled — say

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clayrune-io/clayrune](https://github.com/clayrune-io/clayrune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
