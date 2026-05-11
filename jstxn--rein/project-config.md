---
trigger: always_on
description: Runtime-backed Socratic interview with durable state, clarity scoring, and spec bundle output
---


<Purpose>
rein-interview is the requirements-clarification entry point for REIN when a task is vague, broad, or missing boundaries. It runs a Socratic interview while delegating all persistence, scoring, readiness gates, status reporting, and artifact writing to the `rein interview` runtime commands.
</Purpose>

<Use_When>
- The request is underspecified, risky, or likely to cause misaligned implementation
- The user wants clarification before planning or coding
- You need durable interview state, resumability, or machine-readable handoff artifacts
- You need a spec bundle that can feed `rein-plan`, direct implementation, or further refinement
</Use_When>

<Do_Not_Use_When>
- The request already has concrete file targets and stable acceptance criteria
- The user explicitly asks to skip clarification and execute immediately
- A complete rein-interview spec already exists and the next step should begin
</Do_Not_Use_When>

<Profiles>
- `--quick`: threshold `70%`, max rounds `5`
- `--standard` (default): threshold `80%`, max rounds `12`
- `--deep`: threshold `90%`, max rounds `20`
</Profiles>

<Runtime_Contract>
The runtime owns:
- durable state under `.rein/state/`
- context snapshots under `.rein/context/`
- clarity score computation
- readiness gates
- round ordering
- status display
- transcript/spec/result bundle output

You own:
- asking the next question
- interpreting the user's answer
- deciding the next target dimension
- framing the next question in your own judgment and voice
- producing the final structured summary payload for crystallization
</Runtime_Contract>

<Execution_Policy>
- Ask one question per round, never batch
- Stay intent-first until scope, non-goals, and decision boundaries are explicit
- Use runtime state at every transition point; do not keep the authoritative interview state only in prose
- Show the user current clarity progress after each runtime update
- Do not crystallize until threshold and readiness gates are satisfied, unless the user explicitly accepts the risk
- Do not implement directly inside rein-interview
- Use runtime suggestions for structure, but choose the actual next question yourself
</Execution_Policy>

<Interview_Presentation_Contract>
Use a consistent chat-safe presentation frame for every visible interview turn.

## Layout

Normal round:

```text
[ Interview ]
| Phase: clarifying structure

| Current clarity: N%

| Question
| [one to two short lines]
|
| A. ...
| B. ...
| C. ...
```

Open-question variant:

```text
[ Interview ]
| Phase: clarifying structure

| Current clarity: N%

| Question
| [one to two short lines]
```

Review:

```text
[ Review ]
| Phase: confirming summary

| Current clarity: N%

| Question
| [one to two short lines]
```

Handoff:

```text
[ Handoff ]
| Phase: preparing handoff

| Status: ready for handoff

| Question
| [one to two short lines]
```

## Formatting Rules

- Always preserve blank lines between sections exactly as shown.
- Do not show round counters or max-round counts in the visible interview frame.
- Keep phase names as lowercase phrases.
- For normal rounds and review, show `Current clarity: N%`.
- For handoff, replace clarity with `Status: ready for handoff`.
- Keep the question to one ask, written in one to two short lines.
- If extra context is needed, fold it into the question itself instead of adding another section.

## Header Mapping

- Use `[ Interview ]` for normal rounds.
- Use `[ Review ]` for final confirmation and crystallization.
- Use `[ Handoff ]` for transition into the next workflow.

## Fixed Phase Vocabulary

Use only these phase names:

- `clarifying structure`
- `narrowing scope`
- `resolving tension`
- `confirming summary`
- `preparing handoff`

## Option Rules

- Use `A.`, `B.`, and `C.` options when the user is choosing between clear alternatives.
- Use an open question when structured options would feel forced.
- Do not add a separate reply-hint section for open questions.

## Transition Rules

Use clarity score as guidance, but make the final phase choice with interviewer judgment.

- Start in `clarifying structure`.
- Move into `narrowing scope` when structure is clear and clarity is at least `65%`.
- Move into `resolving tension` when contradiction exists or direction is unstable.
- Tension overrides score-based advancement.
- Move into `confirming summary` when clarity is at least `85%` and no major tension remains.
- Move into `preparing handoff` when the summary is confirmed and the next workflow is identifiable.

## Reopening Rules

- Do not move backward casually between phases.
- Allow reopening only from `[ Review ]`.
- Reopen only for material corrections affecting intent, scope boundaries, tension status, or handoff readiness.
</Interview_Presentation_Contract>

<Command_Sequence>

## Start a new interview

1. Parse arguments:
   - if `--resume <slug>` is present, skip to the resume flow
   - otherwise choose profile from `--quick|--standard|--deep` or default to `standard`
2. Initialize runtime state:

```bash
rein interview init --profile <quick|standard|deep> --idea "<idea>" [--slug <slug>] --json
```

3. Read the returned state and begin the interview.

## Resume an existing interview

1. Load the runtime state:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jstxn/rein](https://github.com/jstxn/rein) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
