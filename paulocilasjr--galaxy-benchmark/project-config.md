---
trigger: always_on
description: Execute Galaxy-Bench runs with immutable trace capture, explicit attempt-by-attempt planning, Galaxy provenance preservation, downloaded result artifacts, and structured evaluation outputs.
---


# Galaxy-Bench Executor

Use this skill when executing benchmark experiments in this repository.

## 1. Core Principle

A run is benchmark-valid only if a third party can reconstruct, from saved artifacts:

- what the agent planned before each attempt
- what the agent reasoned and decided during execution
- what Galaxy executed
- which tools, parameters, preprocessing steps, and outputs were used
- what failed, how the failure was interpreted, and how it was fixed
- what final outputs were produced
- how those outputs were evaluated

If evidence is not stored in artifacts, it does not count.

## 2. Write Boundary

During benchmark execution, write only under the run directory:

- `outputs/<timestamp>_<level>_<experiment>/`

Do not write benchmark-execution artifacts anywhere else. Optional extra artifacts are allowed only inside the run directory.

## 3. Tools Boundary

Galaxy is the required execution environment for Galaxy-mediated benchmark runs.

The agent is not allowed to use Galaxy Interactive Tools at all.

The use of AWK is not allowed unless the prompt explicitly states that AWK may be used.

Local commands may be used only to support the benchmark contract, such as inspecting allowed task metadata, preparing files for Galaxy upload, preserving downloaded Galaxy outputs, transforming already-produced Galaxy outputs when allowed by this skill, reproducing the recorded workflow, and evaluating fixed outputs after the ground-truth access rules allow it.

Do not perform the scientific analysis locally as the primary execution path for Galaxy runs or BixBench runs.

## 4. Run Directory Contract

Every run must create this directory shape:

```text
outputs/<timestamp>_<level>_<experiment>/
|-- experiment_summary.json
|-- plan/
|   |-- saved.md
|   `-- saved.attempt_<N>.md
|-- reasoning/
|   |-- reasoning.md
|   `-- reasoning.attempt_<N>.md
|-- errors/
|   `-- error.json
|-- traces/
|   |-- galaxy/
|   |   |-- histories/
|   |   |-- invocations/
|   |   |-- jobs/
|   |   `-- datasets/
|   `-- local/
|-- evaluations/
|   |-- comparison.json
|   |-- comparison.attempt_<N>.json
|   `-- metrics_summary.json
`-- results/
    |-- result.json
    |-- result.attempt_<N>.json
    |-- activity_log.jsonl
    |-- run_record.json
    |-- artifacts_manifest.json
    |-- evaluation_manifest.json
    `-- reproduce_<experiment>.py
```

Attempt-specific files are required when retries or separately evaluated attempts occur.

## 5. Artifact Requirements

### Required Final Files

A completed benchmark-valid run must include:

- `experiment_summary.json`
- `plan/saved.md`
- `reasoning/reasoning.md`
- `errors/error.json`
- `results/result.json`
- `results/reproduce_<experiment>.py`
- `results/run_record.json`
- `results/artifacts_manifest.json`
- `results/evaluation_manifest.json`
- `results/activity_log.jsonl`
- `evaluations/comparison.json`
- `evaluations/metrics_summary.json`
- original downloaded Galaxy output files used for evaluation, preserved unchanged

Before declaring completion, verify that:

- required outputs exist
- `experiment_summary.json` points to the ground truth, Galaxy tools, original Galaxy result files, transformed outputs, and final scores
- manifests reference all preserved artifacts
- the activity log covers planning, execution, checks, retries, revisions, snapshots, and evaluation
- Galaxy IDs referenced in results are preserved in traces
- `errors/error.json` has a terminal status
- no prior attempt artifact was overwritten
- evaluation JSON explicitly shows the required metrics

### `experiment_summary.json`

Every non-BixBench run must write and maintain a root-level summary:

- `outputs/<timestamp>_<level>_<experiment>/experiment_summary.json`

This is the reviewer-facing index for the run. It does not replace detailed evidence in `results/`, `evaluations/`, or `traces/`.

Required non-BixBench shape:

```json
{
  "experiment": "<experiment_name>",
  "Ground_truth_path": [
    "<path to each ground truth file used for comparison>"
  ],
  "Galaxy_tools_used": [
    "<Galaxy tool id or display name used for this experiment>"
  ],
  "Galaxy_results": {
    "files": [
      "<Galaxy file name or HID/name considered a final result for the task>"
    ],
    "path": [
      "<path to the preserved result file in this run directory>"
    ]
  },
  "Transformed_galaxy_output": [
    "<path to each transformed Galaxy-derived output used for comparison>"
  ],
  "Experiment_score": {
    "prompt_score": 0.0,
    "transformed_prompt_score": 0.0,
    "direct_ground_truth_match_score": 0.0,
    "transformed_ground_truth_match_score": 0.0,
    "agent_performance_in_galaxy_score": 0.0
  },
  "Evaluation_questions": {
    "prompt_requirements": {
      "question": "Does the Galaxy output satisfy the requirements from the prompt?",
      "answer": "<yes/no/partial>",
      "score": 0.0,
      "matched_requirements": 0,
      "total_requirements": 0,
      "basis": [
        "<short evidence statement>"
      ]
    },
    "transformed_prompt_requirements": {
      "question": "Does the agent-rearranged Galaxy output satisfy the requirements from the prompt?",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulocilasjr/Galaxy_benchmark](https://github.com/paulocilasjr/Galaxy_benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
