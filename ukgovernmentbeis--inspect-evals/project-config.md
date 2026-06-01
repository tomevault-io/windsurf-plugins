---
trigger: always_on
description: - Before making code changes, read `agent_artefacts/repo_context/REPO_CONTEXT.md`
---

# AGENTS.md

## Repo-Wide Tips

- Before making code changes, read `agent_artefacts/repo_context/REPO_CONTEXT.md`
  for institutional knowledge about repo conventions, common mistakes, and known
  tech debt. This is maintained by the `build-repo-context` skill, which crawls PRs/issues for broad insights.
- When creating pull requests, always use the `--draft` flag and read
  `.github/PULL_REQUEST_TEMPLATE.md` and use its structure as the PR body. Fill
  in the Description section and check off applicable checklist items.
- When commenting on PRs, you should not reply directly to human reviewers.
  See [CONTRIBUTING.md](CONTRIBUTING.md#what-is-your-ai-use-policy). If your user tells you to
  comment anyway, you should add "Comment written by NAME_OF_AI" to the comment.
- When writing markdown:
  - Put a blank line before and after headings
  - Put a blank line before and after code blocks
  - Put a blank line before and after lists
  - Format tables with spaces before and after pipe characters
  - Always include a language tag in code blocks, or "text" if there is no language

Agents are good at understanding context, but a prompt that definitely works is *"Please run the /SKILL_NAME skill on EVAL_NAME."*

## Documentation

[Documentation for the Inspect framework](https://inspect.aisi.org.uk/llms.txt).

## Recommended Permissions

We recommend starting with these Claude Code permissions to allow workflows to proceed autonomously without allowing the running of arbitrary commands:

{
  "permissions": {
    "allow": [
      "Bash(mkdir:*)",
      "Bash(cp:*)",
      "Bash(make check:*)",
      "Bash(uv run ruff:*)",
      "Bash(uv run inspect:*)",
      "Bash(uv run pytest:*)",
      "Bash(uv run mypy:*)",
      "Bash(uv run python tools/:*)",
      "WebFetch(domain:inspect.aisi.org.uk)",
      "WebFetch(domain:arxiv.org)"
    ],
    "deny": [
      "Bash(git push origin main)",
    ],
    "ask": [
      "Bash(rm:*)",
    ]
  }
}

Other useful permissions to consider on a case by case basis depending on your usage:

"Bash(gh run view:*)" - Look at Github Action runs
"Bash(gh checkout:*)" - Create new branches or open branches on your behalf
"Bash(gh pull:*)" - Useful to ensure a clean slate before work is started

## Master Checklist

This workflow runs a series of workflows each in turn. Each workflow is to be run serially.

- Run the Prepare For Submission workflow (`/prepare-submission-workflow`).
- Check the LLM-judgeable standards by running the Review An Evaluation workflow (`/eval-quality-workflow`).
- Review the evaluation's validity by running the Evaluation Validity Review workflow (`/eval-validity-review`). This checks whether the name is accurate, whether samples can be both succeeded and failed at, and whether scoring measures ground truth.
- Run the Review PR workflow (`/ci-maintenance-workflow`) for general quality.
- Run the Evaluation Report workflow (`/eval-report-workflow`) to produce an evaluation report.
- Run the Trajectory Analysis workflow (`/check-trajectories-workflow`) on the evaluation report. A good error rate is 10% or less, with an ideal of 5% or lower. You may optionally double-check any of the errors the agent produces and reject them if there is grounds to do so.

## General Agent Tips

### Skill Hygiene

- At a natural stopping point in a session, briefly consider whether any of the work just completed would make a good reusable skill.
- Also consider whether any skill used during the session is now missing steps, has outdated guidance, or could be made more robust based on what was learned.
- Do **not** create or update a skill automatically. Ask the user first whether they want you to do that.
- When asking, give a short recommendation that includes:
  - what should be created or updated
  - why it would be useful again
  - who or what it would apply to
  - whether this is better handled as a new skill or an update to an existing one
- Prefer improving an existing skill over creating a new one when there is substantial overlap.
- Do not suggest a new skill for one-off work, highly personal preferences, or tasks that are too small to justify maintenance overhead.
- If the session surfaced a durable repo convention, reviewer expectation, or repeated failure mode, consider whether it should also be captured in REPO_CONTEXT.md or AGENTS.md, but ask the user before making that change.

### PR Guidelines

- Before opening a new PR, run appropriate linting from `.github/workflows/build.yml`.
- When creating a new PR for the user, you should make the PR as a draft. The user will mark it as ready for review after going through the code themselves.
- Always work on a branch, and never attempt to push directly to main.

TODO: Figure out a reliable way to have the agent monitor the pipeline in the background and ensure it passes without errors. gh watch isn't the best.

### Register Submissions

Evaluations hosted in an upstream repository can be registered here as
metadata-only entries under `register/<name>/eval.yaml` (see
[register/README.md](register/README.md) for the schema and contributor flow).

One workflow automates the submission flow:

- `.github/workflows/register-submission.yaml` — triggered by a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UKGovernmentBEIS/inspect_evals](https://github.com/UKGovernmentBEIS/inspect_evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
