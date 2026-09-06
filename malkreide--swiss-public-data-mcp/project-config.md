---
trigger: always_on
description: This repository is the index for the Swiss Public Data MCP servers.
---

# Project conventions for Claude

This repository is the index for the Swiss Public Data MCP servers.
`portfolio.json` is the source of truth; the READMEs, the registry manifests,
the install snippets and the promotion page are **generated** from it and held
to it by `readme-sync.yml` on every pull request. Never hand-edit anything
between `<!-- BEGIN GENERATED: … -->` markers — see `CONTRIBUTING.md` for the
edit-and-regenerate loop.

(No server count here on purpose: it would be a hand-copied number in a file
about not hand-copying numbers. `scripts/coverage_manifest.py --check` prints
the current one.)

## How verification works here

These rules exist because the failures below happened in this portfolio, all
within a few days, and all of them the same shape: **the check ran and checked
nothing.** They are not style. Each one produced a confident, false report.

### Check the result, not the activity

- Never test for a running process with `pgrep -f '<your own pattern>'`. The
  pattern is in the command line you are searching with, so `pgrep` finds
  itself. Test the result instead: `[ -s file ]`, `ls -l`, an exit code.
  *A sweep was reported as "still running" twice, two hours after it finished.*

- Never write `command; echo ok`. The semicolon separates, it does not connect.
  Use `command && echo ok`, or read the state afterwards — the remote ref, the
  file, the API.
  *A push was reported as done while `git` was saying `the remote end hung up
  unexpectedly`.*

- In a pipeline, `$?` is the **last** command's status. `cmd | head` reports
  `head`. Use `${PIPESTATUS[0]}` or drop the pipe.

### A pattern that claims absence must first be proven on a known positive

A grep too narrow finds nothing and looks like a clean result. So does a stub
that never matches.

*Three releases were reported missing because the grep pattern was too narrow.
And an exit-code harness reported failure for every case — including the ones
that pass — because `next(gen, default)` evaluates the default eagerly, so the
stub raised on every call. A working script was nearly "repaired".*

### Run the check that counts, not the one you know

Read what CI actually runs before claiming a local pass. `ruff check` and
`ruff format --check` are two commands; passing the first and reporting "ruff
clean" is a false statement about the second.

The same trap in documents: a check can answer a narrower question than the
one you needed and still come back green.

*A sweep verified that every `CLAUDE.md` carried the shared conventions in
full, and concluded from that the files were correct. The check could not see
part two — the repo-specific half — which the same sweep had just made wrong.
Thirteen files went on telling contributors to install a pinned tool by hand
after the pin had moved. A reviewer found it, not the sweep.*

### Copies that agree still prove nothing if one of them overrides the rest

A guard that compares N places and demands they match reports "clean" the
moment they match. That says nothing about whether the match matters.

*The ruff version was pinned in `pyproject.toml` and again as an install step
in CI. The numbers agreed, and a guard checked that they agreed. But the CI
step ran **after** the dependency install and overwrote it, so a loosened
range in `pyproject.toml` could never turn CI red — it would only have hurt
locally, where nobody expects it. The guard was green throughout.*

Where a place must not exist, check for its **absence**, separately, and
independently of its value. Equality between the survivors is the weaker
claim, and a returning copy satisfies it while defeating it.

### A silent reviewer has four explanations, and only one of them is "clean"

The pull request template carries one line about it: *Codex-Review beantwortet
oder behoben — kein offener Befund beim Merge*. The line assumes a finding could
have existed. Silence does not tell you whether it could.

- **It found nothing** — then it reacts with a thumbs-up and writes no text.
- **The pull request is a draft** — it does not run on those at all.
- **The account's review quota is spent** — then it writes that, and nothing else.
- **The repository has no Codex environment** — then it says that instead, and no
  amount of quota will change it.

From the timeline all four look alike; the difference is in the *form*. A real
review is a **review object**, the quota notice an ordinary **issue comment**.
Those are two different queries, and either one alone answers half the question.
Timing separates them as well — the quota notice came back in ten seconds, a
real review takes three to five minutes. Twenty seconds of silence is not a
result.

*Both quota notices on this repository's own pull request arrived exactly that
way: `get_reviews` returned an empty list while `get_comments` held the refusal.
The pull request was merged with the template line in its description and nobody
having read the diff. The same outage ran twenty-three hours across the
portfolio, and the box was ticked every time.*

A second way to lose the reviewer needs no outage at all: **merging too fast.**
Marking a draft ready is what triggers the review, and the review needs minutes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malkreide/swiss-public-data-mcp](https://github.com/malkreide/swiss-public-data-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
