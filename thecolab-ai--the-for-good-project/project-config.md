---
trigger: always_on
description: > One-page map of how everything fits: [docs/OVERVIEW.md](docs/OVERVIEW.md).
---

# AGENTS.md

> One-page map of how everything fits: [docs/OVERVIEW.md](docs/OVERVIEW.md).
> Git mechanics (branches, worktrees, staying in sync, rules for agents): [docs/GIT-WORKFLOW.md](docs/GIT-WORKFLOW.md).

Instructions for AI agents (Claude Code, and any coding/research agent) working this repo. If you're a person, read [`README.md`](README.md) and [`CONTRIBUTING.md`](CONTRIBUTING.md) instead — though you're welcome to read this too.

You are here to move one issue forward, to a high standard, end to end. Work like a careful researcher, not an eager intern.

## The loop

1. **Pick one unclaimed issue.** Prefer `status: available`. Match it to your strengths. Do **one** issue per branch/PR.
   ```
   gh issue list --label "status: available" --state open --json number,title,labels
   ```
2. **Claim it before you start** (this prevents two agents doing the same work):
   ```
   gh issue edit <n> --add-assignee @me --add-label "status: claimed" --remove-label "status: available"
   gh issue comment <n> --body "Claiming this — starting now."
   ```
   If you can't edit labels (limited token), just comment that you're claiming it.
3. **Read the whole chain.** Open the issue, its parent (`Part of #…`), and any linked findings. Don't re-research something already in `research/findings/`.
4. **Do the work** for the stage (see below), following the method in [`CONTRIBUTING.md`](CONTRIBUTING.md). The method is not optional — cite everything, verify surprises with two sources, mark confidence, say what would change your mind.
5. **Open a PR** on a branch:
   ```
   git checkout -b research/<slug>
   # write your file(s) in the right place
   git add -A && git commit -m "research: <question> (Closes #<n>)"
   git push -u origin research/<slug>
   gh pr create --fill --body "Closes #<n>. Part of #<parent>."
   ```
   **Exception — discover issues:** use `Part of #<n>` instead of `Closes #<n>`
   everywhere. A discover issue is a stream ROOT and must stay open for the
   life of its stream (docs/STREAMS.md); a `Closes` ref would end the stream
   the moment your framing PR merges.
6. **Expect adversarial review.** A reviewer (human or the review agent) will try to *refute* your claims. Respond with evidence, not defensiveness. Fix what's fair.

## No write access? (most contributors)

Most contributors don't have push/label rights on this repo — that's expected. The
loop above still works, you just fork and adjust two steps:

**Claiming (step 2).** `gh issue edit --add-assignee/--add-label` will 403 without
triage rights. Instead just comment so nobody doubles up:
```
gh issue comment <n> --body "Claiming this via Claude Code — starting now."
```
A maintainer's automation keeps the status labels in sync.

**Opening the PR (step 5).** `git push -u origin ...` will 403 without write. Push to
your fork and open the PR across repos (verified working on PR #11):
```
gh repo fork --remote --remote-name fork
git push -u fork research/<slug>
gh pr create --repo thecolab-ai/the-for-good-project \
  --head <your-username>:research/<slug> --base main --fill --body "Closes #<n>. <summary>"
```

**Reviewing as an outside contributor.** You *can* post a full adversarial review
(and an APPROVED / CHANGES_REQUESTED verdict) on a PR you didn't author, even without
write access — `review_work.sh` does exactly this and records it. You *can't* set the
`for-good/adversarial-review` status check (that needs write), but you don't need to:
a maintainer runs `merge_ready.sh`, which reads your recorded review, checks it against
the trust model (whitelist or earned credit), and merges if it qualifies. So your
review still counts toward the gate — it's just validated + merged by a maintainer.
One hard rule: agents never apply or remove the `review: human-only` label — a PR
carrying it is reviewed and merged by humans; leave it alone.

**When only a maintainer can act** ([ADR-0009](docs/adr/0009-maintainer-escalation-handoff.md)).
Some steps need write access no matter how you
route them: syncing status labels the automation missed, pushing rework to an
upstream PR branch, dismissing a stale review, running `merge_ready.sh`, or anything
touching `review: human-only`. Don't stall silently, don't retry the 403, and never
work around the permission — hand it off so a maintainer can act in one paste:

1. **Comment on the affected PR/issue** stating exactly what's needed, with
   copy-paste commands. If it's rework you couldn't push, push the commit to your
   fork first and include the `git fetch <your-fork> <branch> && git merge --ff-only
   FETCH_HEAD && git push` line so it's a one-liner to land.
2. **If it spans several threads** (or risks getting lost), open a tracking issue
   titled `maintainer: <what's needed>` that tags a maintainer (currently
   @adam91holt) and lists each action with links + commands — see
   [#111](https://github.com/thecolab-ai/the-for-good-project/issues/111) for the
   shape.
3. **Say who you act for** — sign comments "posted by an agent on behalf of
   @<your-human>" so the trust model stays legible.

The escalation *is* the handoff: once it's posted, move on to other available work
rather than waiting.

## What each stage needs from you


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecolab-ai/the-for-good-project](https://github.com/thecolab-ai/the-for-good-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
