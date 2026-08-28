---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`dsh-crew` is a **plugin for DeepSeek Harness (dsh)**, not an application. Nothing here runs on its
own. dsh loads the modules in `host/` and the agent preset in `preset/crew/`, and the result is a
"crew": your dsh session becomes a product manager (PM) that starts role agents (architect, engineer,
the two engineers of a paired task, reviewers, QA, researcher) as its direct children.

There is no build step and no bundler. The package ships plain ES modules (`"type": "module"`).

## Commands

```sh
npm test                            # every check below, in order
node tools/verify-guard.mjs         # git-guard rules, replayed against fake commands
node tools/verify-pm-write-guard.mjs# the PM write guard, against fake write/edit paths
node tools/verify-rule-guard-map.mjs# the rule→guard map does not lie
node tools/verify-jobs.mjs          # the unfinished-job notice, using throwaway job folders
node tools/verify-mount.mjs         # package shape, preset shape, role table, real mount
node tools/verify-preset-install.mjs # installing and upgrading the crew preset
bash qa/run-all.sh             # every crew job's QA cases, past and present
node tools/verify-tasks.mjs         # the Verdicts line of every task file under docs/tasks/
```

Every check runs against temporary folders and a throwaway `DSH_HOME`. None of
them may read or write the real `~/.dsh` — keep it that way when adding cases.

Run one check on its own by calling its file directly — that is the "single test" here.

`npm test` runs every check below in order: the project checks first, then
`bash qa/run-all.sh`, then `node tools/verify-tasks.mjs`. QA's cases and the Verdicts gate are
part of the default test command and not things you have to remember. `npm test` is what CI runs:
`.github/workflows/test.yml` runs it on **every push**; `.github/workflows/publish.yml` runs on a
`v*` **tag** only and runs `npm test` again before it publishes — a release never trusts an earlier
push's green. Expect `npm test` to get slower as jobs add cases; when that starts to hurt, split it
into a fast check and a full one rather than dropping the cases. `test.yml` checks out with
`fetch-depth: 0` on purpose: some QA cases read this repository's own commits, and the default
shallow clone has no history.

`verify-tasks.mjs` is the last check, and it reads no code — it reads the task files under `docs/tasks/`.
One file per task: a `T-<n>.md` file whose top heading `# T-<n> — …` declares the same id is one
task section; `README.md` and any other file are not read, and an empty directory is red. A
section turns the check **red** when:

1. it has no `- **Verdicts**：` line, or more than one;
2. any of the four values `code`, `security`, `qa` and `doc` is missing;
3. a `not run` or `skipped` value carries no reason of its own after the dash;
4. a `changes needed` value names no `T-<number>` to carry the fix.

Every run also prints the totals out loud — how many values are `not run` and how many are `skipped` across all task sections.
**Passing is not the same as clean**: the check proves the Verdicts line was written and every skip
carries a reason, and it **cannot** prove a review happened — a `code: pass` typed by the PM passes
it. Nothing automated can close that hole. It exists because the PM of this repository's own job
skipped code review on about 20 tasks and doc review on most of the job, nothing went red, and
nobody knew until the user asked (`docs/decisions/crd/0011-verdicts-gate-in-npm-test.md`).

`verify-mount.mjs` has two levels. `@deepseek-ai/dsh-tool-subagent` cannot be installed from the
public npm registry (its peer `@deepseek-ai/dsh-tasks` is not published), so on a plain machine the
check **skips** the role-tool half out loud. **CI is such a plain machine** — neither workflow runs
`npm install`, so green CI means "everything a public runner can check", not "everything". To get the
full check locally, link dsh's own copy once:

```sh
mkdir -p node_modules/@deepseek-ai
ln -s ~/.dsh/profiles/node_modules/@deepseek-ai/dsh-tool-subagent \
      node_modules/@deepseek-ai/dsh-tool-subagent
```

That link already exists in this working copy. Never add a real dependency on that package — it is a
`peerDependencies` entry on purpose.

Releases: put the new version's section at the top of `CHANGELOG.md` (newest first, plain
English, what a user would notice). **On the release day, replace `— unreleased` in that heading
with the date.** Miss that step and the release fails:
`qa/T-81/case-01-changelog-order.mjs` reds when the top section is still marked `unreleased`
while `package.json` already holds that version, and `npm test` runs inside the tag's own workflow.
Then bump `version` in `package.json`, commit, push `main`, wait for CI to go green, and push the
matching `v*` tag.
Only the tag triggers `.github/workflows/publish.yml`. The workflow fails loudly if the tag and
`package.json` disagree. Auth is npm trusted publishing (OIDC) — there is no secret to set.
**After the run, open the GitHub Releases page and read what landed there.** Nothing else checks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stuarthu/dsh-crew](https://github.com/stuarthu/dsh-crew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
