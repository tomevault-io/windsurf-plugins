---
trigger: always_on
description: <!-- SPECKIT START -->
---

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
at specs/045-clickable-file-links/plan.md
<!-- SPECKIT END -->

## Verifying done-ness

**`npm run gate` is the only thing that establishes work is done.** Not a green unit run, not a
passing spec, not "the tests I changed pass" — those are progress, and reporting one as done-ness is
the specific mistake this rule exists to stop.

It runs the eight gating stages in CI's order, fail-fast: **lint → typecheck → build → unit →
component → integration → contract → e2e**. Component sits fifth, immediately after unit and
before the OS-heavy layers, on purpose — it is the
second-cheapest layer (jsdom, no app, no daemon, no shell) and it now carries assertions that used to
cost an Electron launch each, so running it after the OS-heavy layers would spend minutes to learn
something available in seconds. It prints one line per stage, stops at the first failure, and clears
the app/daemon/pty-agent/Playwright processes a run leaves behind — on success, on failure, and on
Ctrl+C.

**It runs on a GitHub-hosted runner, not here.** Dispatch it against a ref and wait:

```bash
gh workflow run gate.yml --ref <branch>
gh run watch <run-id> --exit-status                       # one blocking watch, expect ~35 min
gh run view <run-id> --json status,conclusion --jq '"\(.status)/\(.conclusion)"'
```

It ran on a self-hosted Windows VM for a day and was moved back to `windows-2022`; `gate.yml`
carries the measurements. The short version: hosted is about twice as fast on this workload, free on
a public repo, and needs no maintaining. The one thing it CANNOT do is run the `skipIfElevated()`
specs — hosted runners are administrators with UAC disabled, so there is no filtered token to drop
to. `admin-reminder.reporter.ts` prints how many were skipped on every run, so that gap is reported
rather than assumed.

**The second command is not belt-and-braces.** `gh run watch` detaches early and its exit code then
lies both ways — observed returning `0` for a run that concluded `failure`, and `1` for one still
in progress. Take the verdict from `run view`; if `status` is not `completed`, the watch gave up and
the run is still going.

Running it locally is not a faster route to the same answer — it pins every core for the better part
of an hour and eventually exhausts the interactive desktop heap. See *Where a test is allowed to run*
in the `throng-testing` skill for what the workstation is still for.

First green run, for the record: **34m06s**, all eight stages, E2E 181 parallel + 333 serial, zero
failed and zero flaky — <https://github.com/Bidthedog/throng/actions/runs/34115448861> at
`9c931345`.

Three rules about using it:

- **Fail-fast means stop, fix, re-run — not read on.** When a stage fails the gate cancels the run.
  Fix that failure before anything else, using the **running-tests** skill to re-run only what failed
  and **throng-testing** when the failure is an E2E flake rather than a defect. Do not queue up more
  work on top of a red gate.
- **Never bypass the E2E stage to make the gate finish sooner.** E2E is ~18 minutes locally (measured
  2026-08-20 at 207 spec files / 548 declarations; see `docs/testing.md`), and that expense is
  exactly why it is inside the gate rather than optional:
  the cheap stages run first precisely so the expensive one is only ever reached by code that has
  already earned it. Running the individual `npm run test:*` scripts while iterating is fine and
  expected — it is claiming *done* off the back of them that is not.
- **A green gate goes stale the moment you edit** — and a remote gate is triggered against a REF, so
  it was only ever evidence about that *commit*, never about the working tree. Quote the run URL and
  the SHA when reporting done, not just the stage summary, and re-run if anything changed after it.
- **The evidence is the TREE, not the SHA. A rewrite that changes no content needs no new gate.** A
  rebase, a squash, a regrouping of commits, a reworded commit message — none of them change what
  gets built and tested, so a gate that was green on the old tip is still evidence about the new one.
  Prove it rather than asserting it, with the check the rewrite already owes you:

  ```sh
  git diff --stat <green-gate-sha> HEAD     # empty = identical content, the gate still holds
  git rev-parse HEAD^{tree}                 # …or compare tree SHAs directly
  ```

  Empty diff → **do not re-run the gate.** Say which SHA was green, that the tree is unchanged since,
  and quote the original run URL; that is a complete done-ness claim. Non-empty → the gate is stale
  in the ordinary way and the rule above applies.

  This exists because the opposite reflex is expensive and looks rigorous: `branch-sync` rebuilt 181
  commits into 7, the green SHA stopped being an ancestor, and re-running cost ~38 runner-minutes to
  re-test a byte-identical tree. **GitHub's own required checks are a different thing** — they are
  keyed to the head SHA, they re-run on a force-push whatever you think, and they gate the merge.
  Nothing here skips those; this rule is only about not *dispatching* `gate.yml` yourself.

## Formatting is ESLint's job — never run Prettier here


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bidthedog/throng](https://github.com/Bidthedog/throng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
