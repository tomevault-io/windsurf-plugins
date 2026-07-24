---
trigger: always_on
description: Repo-local reference docs that Claude should consult when the task
---

# Instructions to work with the code base

## Reference docs for Claude

Repo-local reference docs that Claude should consult when the task
touches the relevant area:

- `.claude/docs/gspread.md` — Google Sheets integration test setup. **Read
  this before attempting any Google Sheets automation via the
  Claude-in-Chrome plugin**: in the `tradingstrategy.ai` Workspace
  environment we've tested, sharing a sheet with a service account
  cannot be completed by Claude-in-Chrome and must be performed
  manually by the operator. Other Workspace orgs may behave differently.
- `.claude/docs/agent-tricks-and-troubleshooting.md` — Codex CLI and
  Claude CLI usage patterns, including cross-agent review commands,
  streaming Claude review output, and common failure modes.
  **Read this before invoking Claude CLI or Codex CLI for any review,
  sanity check, plan review, PR review, or one-off agent run.**
  Follow its invocation patterns for streaming output, tool restrictions,
  timeouts, no-tools plan reviews, and silent or hanging agent runs.

## Agent review workflows

- **Blocking requirement: before running any `claude`, `claude -p`, `claude ultrareview`, `codex`, or `codex exec` command, read `.claude/docs/agent-tricks-and-troubleshooting.md` in the current session.** Do not invoke either CLI until you have checked the repo-local guidance.
- For plan reviews with Claude CLI, default to the no-tools inline review pattern from `.claude/docs/agent-tricks-and-troubleshooting.md` after the primary agent has inspected the relevant code. Only use a grounded tool-using review when fresh repository inspection is actually required.
- For code and PR reviews with Claude CLI, scope the request to correctness bugs, behavioural regressions, missing tests, security or money-movement risks, and repository instruction compliance. Ask for findings first with file:line references and residual risks.
- For long Claude CLI reviews, use streaming output (`--output-format stream-json --verbose`) and a wall-clock timeout. If a grounded review produces no output after roughly one minute, stop it and switch to a smaller no-tools or file-group review unless repository inspection is strictly required.
- Do not paste huge diffs into Claude prompts. Make Claude inspect `git status --short`, `git diff --name-only`, and targeted hunks, or provide only the plan text for no-tools plan reviews.
- For non-interactive Codex reviews, use `codex exec --json` in read-only mode as described in `.claude/docs/agent-tricks-and-troubleshooting.md`. Plain text mode can buffer output and look hung.
- Before trusting any external-agent "no findings" result, verify it reviewed the correct worktree and non-empty diff.

## Skills

Repo-local skills live as folders under `.claude/skills`.

When a task matches one of the folder names, open the corresponding `SKILL.md` first and follow it before doing ad hoc exploration.

Skill discovery rules:

- If the user mentions a skill by name, use it.
- If the task clearly matches one of the skill directory names above, use that skill even if the user did not mention it explicitly.
- Start by reading only `.claude/skills/<skill-name>/SKILL.md`.
- If the skill references extra files, open only the files needed for the current task.
- Prefer scripts, templates and checklists referenced by the skill over re-creating the workflow manually.

## English

- Use UK/British English instead of US English
- Say things like `visualise` instead of `visualize`
- Always spell `onchain` without a hyphen; correct `on-chain` to `onchain`
- For headings, only capitalise the first letter of heading, do not use title case

## Installing dependencies

Install dependencies with all required extras:

```shell
poetry install -E data -E test -E docs -E hypersync -E ccxt -E cloudflare_r2 -E duckdb
```

## Running Python scripts

When running a Python script use `poetry run python` command instead of plain `python` command, so that the virtual environment is activated.

```shell
poetry run python scripts/logos/post-process-logo.py
```

## Running tests

If we have not run tests before make sure the user has created a gitignored file `.local-test.env` in the repository root. This will use `source` shell command to include the actual test secrets which lie outside the repository structure. Note: this file does not contain actual environment variables, just a `source` command to get them from elsewhere. **Never edit this file**.

If `.local-test.env` is missing in a git worktree when pytest needs to be run, do not give up. First follow the git worktree instructions below and copy `.local-test.env` from the main repository checkout into the current worktree root. Only ask the user to prepare `.local-test.env` if it is also missing from the main repository checkout or cannot be found.

To run tests you need to use the installed Poetry environment, with given environment secrets file.

To run tests use the `pytest` wrapper command:

```shell
source .local-test.env && poetry run pytest {test case name or pattern here}
```

Always prefix pytest command with relevant source command,
otherwise the test cannot find environment variables.

Avoid running the whole test suite as it takes several minutes. Only run specific test cases.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tradingstrategy-ai/web3-ethereum-defi](https://github.com/tradingstrategy-ai/web3-ethereum-defi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
