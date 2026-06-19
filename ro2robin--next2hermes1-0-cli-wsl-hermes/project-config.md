---
trigger: always_on
description: Run a WSL-native Hermes coding workflow that uses GitHub Copilot CLI as the primary executor and Codex CLI as the fallback executor, with fixed wrappers, structured outputs, and file-based auditing.
---


# WSL Copilot-Primary Workflow

Use this skill when:
- Hermes is running inside WSL
- the repository to operate on is also inside WSL, ideally under `/home/rorobin/code/...`
- GitHub Copilot CLI should be the default executor
- Codex CLI should only take over when Copilot fails, times out, returns unusable output, or the user explicitly asks for a second opinion

This skill is for the **native WSL path**, not the Windows-through-`cmd.exe` workaround. In the current environment, both CLIs are available natively in WSL:
- `copilot` → `/home/rorobin/.local/bin/copilot`
- `codex` → `/home/rorobin/.local/bin/codex`

## Workspace layout

Expected workflow directory:

```bash
/home/rorobin/ai-workflow/
├── bin/
│   ├── build_copilot_prompt.sh
│   ├── run_copilot_main.sh
│   ├── run_codex_task.sh
│   ├── run_agent_orchestrator.sh
│   ├── run_workflow_reply.sh
│   ├── format_orchestrator_reply.py
│   └── parse_agent_result.py
├── logs/
├── tests/
└── tmp/
```

Meaning:
- `bin/` stores wrappers, the parser, the user-reply formatter, and the one-key entry script
- `logs/` stores runtime logs
- `tests/` stores lightweight script-level verification for the orchestration flow
- `tmp/` stores prompt files, final outputs, and intermediate artifacts

## Required behavior

### 1. Keep execution inside WSL
Do not bounce the main flow into Windows commands when the repo and tools are already available in WSL.

Preferred repo root:

```bash
/home/rorobin/code/<repo>
```

Avoid defaulting to `/mnt/c/...` for normal project work unless the user explicitly wants a Windows-backed repo.

### 2. Check repository validity first
Before invoking either agent, verify:
1. `repo_dir` exists
2. `repo_dir/.git` exists
3. the repo is on a WSL-native path if possible

If any check fails, stop and report the problem instead of attempting the agent run.

## Task classification

Classify every request into one of these four types before execution:

### `analyze`
Read-only analysis.
Use for:
- root-cause investigation
- code explanation
- error analysis
- repair suggestions without editing

### `fix`
Minimal necessary change plus the most relevant validation.
Use for:
- bug fixes
- config fixes
- small targeted repairs

### `implement`
New feature or broader code addition.
Use for:
- new modules
- new endpoints
- new pages or capabilities

### `review`
Review only, no edits.
Use for:
- change review
- diff inspection
- risk assessment

## Preferred entrypoint: orchestrated workflow

For normal Hermes execution, prefer the orchestration wrapper rather than manually chaining the lower-level wrappers:

```bash
~/ai-workflow/bin/run_agent_orchestrator.sh <repo_dir> <task_type> <user_request_file> [copilot_model] [fallback_mode] [codex_model]
```

Defaults:
- `copilot_model`: `auto`
- `fallback_mode`: `allow`
- `codex_model`: `gpt-5.4`

Fallback modes:
- `allow`: retry Copilot once, then allow Codex fallback
- `no-fallback`: retry Copilot once, then fail without running Codex

The orchestrator currently does all of the following:
1. validates repo path, `.git`, task type, and dependency scripts
2. archives the user request into `tmp/`
3. runs Copilot up to two attempts
4. validates that the final output is non-empty, contains the required section headers, and produces a non-empty parsed summary
5. builds a fallback prompt and runs Codex when policy allows
6. reuses the timestamped Codex output files emitted by `run_codex_task.sh` directly, avoiding duplicate fallback archiving inside the orchestrator
7. emits machine-readable result lines such as:
   - `STATUS=...`
   - `TASK_TYPE=...`
   - `EXECUTOR_USED=...`
   - `FALLBACK_OCCURRED=...`
   - `COPILOT_ATTEMPTS=...`
   - `FINAL_MESSAGE_FILE=...`
   - `PARSED_JSON_FILE=...`
   - `LOG_FILE=...`
   - `EVENTS_FILE=...`
   - `REASON=...`
   - `MISSING_DEPENDENCY=...` (when applicable)
   - `ORCHESTRATOR_LOG=...`

Use the lower-level wrappers directly only when you intentionally want to bypass orchestration.

## One-key entrypoint

For the simplest end-to-end usage, prefer:

```bash
~/ai-workflow/bin/run_workflow_reply.sh [--json] <repo_dir> <task_type> <user_request_file> [copilot_model] [fallback_mode] [codex_model]
```

This script:
1. runs `run_agent_orchestrator.sh`
2. saves the machine-readable orchestrator output into `tmp/workflow_reply_<timestamp>_<pid>.out`
3. passes that file into `format_orchestrator_reply.py`
4. in default mode, prints the final Chinese user-facing reply to stdout
5. in `--json` mode, prints a JSON object containing the final `reply` plus key metadata such as `status`, `task_type`, `executor_used`, `reason`, and `orchestrator_output_file`
6. when `PARSED_JSON_FILE` exists and is readable, `--json` mode also inlines commonly used parsed fields such as `summary`, `root_cause`, `changed_files`, `commands_run`, `test_result`, `risks`, and `next_step`

Use `--json` when Hermes also needs structured metadata without separately re-reading the saved orchestrator output file.
This is especially useful when Hermes wants both a ready-to-send `reply` and direct access to parsed fields like `summary`, `root_cause`, and `test_result`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ro2robin/Next2Hermes1.0-Cli-Wsl-Hermes](https://github.com/Ro2robin/Next2Hermes1.0-Cli-Wsl-Hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
