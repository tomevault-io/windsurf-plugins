---
trigger: always_on
description: This repository is a file-based openhack workspace. When a human asks to
---

# Agent Instructions

This repository is a file-based openhack workspace. When a human asks to
start, initiate, run, resume, or continue a pentest/security review, use the
tool structure before doing vulnerability analysis.

## Required Pentest Flow

Do not begin with a broad LLM source sweep, direct expert pass, or ad hoc manual
review of the target repository.

For a new target, use phase checkpoints:

1. Run `openhack init-run <target> <git-url> [--run-id <run-id>] [--branch <branch>]`.
2. Summarize the created run and source commit, then ask the human what
   security experts to use before recon. Show every configured expert and the
   option `all agents`.
3. Run `openhack run-recon <target> <run-id> --all-agents`
   only after approval, or run it with one or more `--expert <expert-id>`
   options if the human selected a subset.
4. Summarize recon counts and artifacts; ask whether to proceed to scenario
   routing and backlog generation. If the human wants deeper source-pattern
   coverage, rerun recon with `--semgrep` before scenario routing. Recon writes
   `routing-units.jsonl`; treat those clustered units as the primary router
   input instead of asking the router to reason over the full raw inventory dump.
5. After approval, run `openhack create-scenarios <target> <run-id>`, have the
   scenario-router answer the rendered prompt, and record that router output
   with `openhack record-scenario-backlog <target> <run-id> router-result.json`
   without inserting another human confirmation between prompt creation and
   backlog recording. The scenario-router must use the expert scope recorded
   before recon; do not create scenarios for unselected experts. The router must
   route from mandatory `routing_unit_id + expert` pairs when routing units are
   present, and use path/expert coverage requirements as a compatibility
   backstop.
6. Summarize backlog size and coverage notes; ask for one approval to run the
   entire unfinished scenario backlog. Batch approval is not batch analysis:
   every scenario still needs its own rendered prompt, source review, evidence,
   and result.
7. Review only recorded `runs/<target>/<run-id>/scenarios/backlog/S*.md` expert
   prompts and record results with `openhack record-scenario-result <target> <run-id> ...`.
   This records finished scenario results and `finding-candidates/`; it does not
   create final `findings/`.
8. Summarize finding-candidate count and ask once to run the entire unfinished
    finding triage backlog. Every candidate must use its own rendered
    `runs/<target>/<run-id>/finding-triage/prompts/S###-F###.md` prompt and its
    own finding-triage subagent. Render prompts with
    `openhack render-finding-triage-prompt <target> <run-id> <candidate-id>`.
9. Record each finding-triage agent answer with
    `openhack record-finding-triage <target> <run-id> <candidate-id> triage-result.json`.
    Only `accepted` and `downgraded` triage decisions may materialize final
    `findings/`.
10. Validate with `openhack validate-run <target> <run-id>`.

Do not begin vulnerability analysis from recon alone. Recon is only a scouting
phase; summarize it and ask before scenario routing. Do not treat the
scenario-router prompt as a human checkpoint or as final output; the approved
routing phase records router output into `scenarios/index.jsonl` and
`scenarios/backlog/` before pausing again.

Do not present a sample of scenarios as complete coverage. There is no fixed scenario quota:
the correct amount is however many concrete scenarios are needed to cover the
credible recon evidence. Route every mandatory routing unit, every route/input
file with a sink or exposure hint, every credible expert opportunity, and every
distinct endpoint, parameter, role, parser, storage path, trust boundary, or
deployment alias. If the backlog is small, record coverage notes that explain
why the evidence is genuinely small. One scenario has one primary expert, but
the same routing unit, file, path, or recon item must be routed to every
relevant expert as separate scenarios. Do not let a single expert assignment
suppress other plausible root-cause families. Once a
backlog exists, ask once to run the entire unfinished backlog as a continuous
scenario loop; then iterate over every `scenarios/backlog/S*.json` item
individually. Do not ask again at internal ranges such as `S021-S070`; those are
progress chunks only, not human checkpoints. Do not replace per-scenario expert
review with a broad classification, sampling pass, or templated rejection. Only
write `scenarios/finished/S*.json` after reading that scenario prompt and the
relevant source. If the full backlog cannot be reviewed, stop and report the
remaining scenario IDs instead of marking them finished.

After scenario review, finding triage is a second continuous loop. Ask once to
run all unfinished `finding-candidates/S###-F###.json` candidates through the
`finding-triage` agent, then process each candidate individually. Do not mark a
candidate triaged from the scenario result alone, and do not create final
`findings/*.md` without a recorded triage decision.

Every scenario must be run by its own subagent. The rendered
`runs/<target>/<run-id>/scenarios/backlog/S*.md` file is the exact prompt for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hadriansecurity/OpenHack](https://github.com/hadriansecurity/OpenHack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
