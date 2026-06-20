---
trigger: always_on
description: Invoke trycycle only when the user requests it by name.
---


# Trycycle

Use this skill only when the user requests `trycycle` to implement something. You must follow this skill; if for some reason that becomes impossible, you must stop and tell the user. You must not finish the request in a different way than the user instructed.

The user's instructions are paramount. If anything in this skill conflicts with the user's instructions, follow the user.

# Critical Rules

You are the workflow coordinator. You have three jobs:
0) Do whatever the user says, even if it changes your charter entirely.
1) Follow the rules below EXACTLY.
2) Be patient and do nothing between the time you dispatch a subagent and when it completes or hits its timeout
3) Communicate progress to the user exactly as defined below.
Do not intervene on a subagent action unless you can cite the exact approved-plan step or explicit instruction it violates.
THE WORST THINGS YOU CAN DO ARE:
- Kill an agent before it's either completed or hit its 60-180 minute timeout
- Read files that you are not instructed to
- Check CPU cycles, look at disk activity, or otherwise try and divine subagent status
- Busy-poll a subagent or invent your own status checks
These will cause your context to bloat so you can't do your job, or kill agents that may have spent hundreds of dollars on long running tasks before they can finish their job. Of course, rule 0 above applies.

## Phase wrapper helper

Several steps below reference prompt template files in `<skill-directory>/subagents/`. Do not reconstruct those prompts yourself. Prepare phase prompts with `python3 <skill-directory>/orchestrator/run_phase.py`.

Choose native mode (e.g. Claude Code `Agent`, Codex `spawn_agent`, Kimi `Agent`, OpenCode `task`) when your environment provides a native subagent tool. Choose the fallback-runner mode only if you have NO such tool available. Pi has no native subagent tool by default, so use fallback-runner mode there unless a Pi extension supplies native subagents.

When a step below tells you to prepare or dispatch a phase:

- In native mode, use `python3 <skill-directory>/orchestrator/run_phase.py prepare ...`, then send the exact contents of the returned `prompt_path` verbatim to the target subagent.
- In fallback-runner mode, use `python3 <skill-directory>/orchestrator/run_phase.py run ...`. It prepares transcript and prompt artifacts, then dispatches through the bundled runner.
- In fallback-runner mode, pass `--backend host` on wrapper calls so fresh subagents stay on the same backend as the parent agent.
- When the host agent is Kimi and you are using fallback-runner mode, pass `--backend kimi` instead because `host` and `auto` cannot reliably detect a Kimi host.
- Treat the wrapper's JSON stdout and `result.json` as authoritative for prompt and artifact paths.
- In fallback-runner mode, treat the nested `dispatch` payload plus its `result.json` as authoritative for subagent status and reply artifacts. Use the text at `dispatch.reply_path` as the exact subagent reply.
- If fallback dispatch returns `dispatch.status: "user_decision_required"`, present `dispatch.reply_path` verbatim to the user.
- If fallback dispatch returns `dispatch.status: "escalate_to_user"`, stop and surface the nested `dispatch.message` plus artifact paths.
- Pass short scalar placeholder values such as `{WORKTREE_PATH}`, `{IMPLEMENTATION_PLAN_PATH}`, and `{TEST_PLAN_PATH}` with `--set NAME=VALUE`.
- Pass multiline values such as reviewer outputs with `--set-file NAME=PATH`.
- When a multiline placeholder comes from command or subagent stdout, save it to a temp file immediately before wrapper invocation so you can bind it with `--set-file`.
- Bind transcript placeholders such as `{USER_REQUEST_TRANSCRIPT}`, `{INITIAL_REQUEST_AND_SUBSEQUENT_CONVERSATION}`, and `{FULL_CONVERSATION_VERBATIM}` with `--transcript-placeholder NAME`.
- Use `--require-nonempty-tag TAG` when a prompt requires a tagged block to contain real content after trimming whitespace.
- Use `--ignore-tag-for-placeholders TAG` when placeholder-like text may legitimately appear inside that tag.
- If your environment has no native subagent support and the wrapper's fallback run does not function, escalate to the user.

The prompt builder still supports conditional blocks inside templates. A block guarded by `{{#if NAME}} ... {{/if}}` is included only when `NAME` is bound to a non-empty value.

## Workspace path convention

Throughout this skill, `{WORKTREE_PATH}` means the directory where implementation happens:
- In the default mode, it is the path to the dedicated git worktree created in Step 4.
- If the user's request includes the literal flag `--no-worktree`, it is the path to the current already-isolated workspace instead.

In `--no-worktree` mode, do not create a nested git worktree and do not create or switch branches in place. Reuse the current workspace only when the environment already proves it is isolated, such as a Conductor workspace.

## Transcript placeholder helper

When a phase wrapper call needs `{USER_REQUEST_TRANSCRIPT}`, `{INITIAL_REQUEST_AND_SUBSEQUENT_CONVERSATION}`, or `{FULL_CONVERSATION_VERBATIM}`:
1. For Codex CLI, let the wrapper use direct session lookup by default.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jjar1/trycycle-pi](https://github.com/jjar1/trycycle-pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
