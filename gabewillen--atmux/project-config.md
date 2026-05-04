---
trigger: always_on
description: - Use plain `atmux ...` commands; do not prefix them with inherited session environment.
---











<atmux>
# Role
- ROLE: implementer

# atmux Rules
- Use plain `atmux ...` commands; do not prefix them with inherited session environment.

# Managed Agent Rules
- ALWAYS acknowledge manager messages quickly with a short plan.
- ALWAYS send a message to your manager when stuck or after completing any task.
- ALWAYS message your manager with `atmux send --to manager "..."`.
- ALWAYS coordinate with peer agents using `atmux send --to <agent> "..."`.
- ALWAYS check `atmux agent list --all --status` before creating new agents.
- ALWAYS reuse idle capable agents before creating new ones.
- ALWAYS spawn agents to decompose your todos if necessary.
- ALWAYS use `--reply-required` when a manager decision is needed.
- NEVER poll agent panes unless absolutely necessary.
- NEVER silently change scope; ask your manager first.
- NEVER report task completion without validation evidence.
- NEVER leave blockers unreported; escalate immediately.

# atmux help
## create
Usage:
  atmux agent create <name> --role <role> --intelligence <0-100> [--team <team>] [--adapter <adapter>] [--shared-worktree] [--task --description <desc> --todo <todo>...] [-- <adapter-args...>]
  atmux team create <name>
  atmux issue create --title <title> [--description <description>] [--todo <todo>...] [--repo <repo>]

Description:
  Unified create entrypoint for agents, teams, and issues.
  For agents, --team defaults to ATMUX_TEAM when set (for example after `atmux team create <name>` in a tmux session).

## list
Usage:
  atmux team list
  atmux session list
  atmux agent list [--all] [--status]
  atmux issue list [--repo <repo>]
  atmux message list [--unread]

Description:
  Listings are implemented by scripts under bin/(atmux)/(list)/.

## send
Usage:
  atmux send --to <name|session> [--reply-required] [--interrupt] "message"

Description:
  Send XML messages to a single agent or every agent in a team. Without
  --interrupt, the message is queued and delivered when the receiving agent
  is at its idle prompt.
  Resolution order for --to:
    1) Team session/name
    2) Agent session/name
  --interrupt  Hard interrupt: send the adapter's abort key sequence
               (`submit_keys.interrupt` in the manifest) to stop the current
               operation, then submit the message. Use sparingly — this aborts
               whatever the agent is doing.

Examples:
  atmux send --to planner "run tests"
  atmux send --to platform --reply-required "status check-in"
  atmux send --to worker --interrupt "stop, that's wrong"

## message
Usage:
  atmux message read <id> [--repo <repo>]
  atmux message list [--unread]

Description:
  Read or list filesystem-backed messages.
  Messages are stored at: ~/.atmux/messages/<repo>/<id>/

## schedule
Usage:
  atmux schedule (--interval <duration> | --once <duration>) [--no-detach] --notification <text>
  atmux schedule (--interval <duration> | --once <duration>) [--no-detach] -- <command> [args...]

Description:
  Schedule a future or repeating action. Use `--notification` to queue an
  ATMUX notification to the current session, or use `-- <command...>`
  to run any command after the delay.

  Notification mode:
    - Always targets the current agent/session.
    - Use this for self reminders, ticks, and status checks.

  Command mode:
    - Runs the provided command in the current environment.
    - Only schedule `atmux send` when the target is another agent or team:
      `atmux schedule --once 10m -- atmux send --to worker "status check"`
    - Never schedule `atmux send --to <self>`; use `--notification` instead.

  --no-detach  Run in the foreground (blocking). By default, the scheduled
               task runs in a detached tmux window and the command returns
               immediately.

Durations:
  Supports integer values with optional unit suffix:
    ms  milliseconds
    s   seconds
    m   minutes
    h   hours
    d   days
  If no suffix is provided, seconds are assumed.

Examples:
  atmux schedule --interval 30m --notification "check on long-running jobs"
  atmux schedule --once 45s --notification "tick"
  atmux schedule --once 45s -- atmux send --to atmux-myrepo-worker "follow up"

## issue create / issue assign
Usage:
  atmux issue create --title <title> --assign-to <agent|session> [--description <description>] [--given <context>] [--when <action>] [--then <outcome>] [--todo <todo>]... [--repo <repo>]
  atmux issue assign <id> --to <agent|session> [--repo <repo>]

Description:
  Assign work using filesystem issues.
  - `issue create --assign-to`: creates a new issue, then assigns it.
  - `issue assign`: assigns an existing issue id.

## issue comment / pr comment
Usage:
  atmux issue comment <id> "message" [--repo <repo>]
  atmux pr comment <id> "message" [--repo <repo>]

Description:
  Add a comment to a filesystem issue or pull request.
  Notifies watchers, assignee, and assigner.

## capture
Usage:
  atmux agent capture <name|session> [--lines <n>]
  atmux team capture <name|session> [--lines <n>]
  atmux agent capture --all [--lines <n>]

Description:
  Capture tmux pane output for agents or team sessions.

Examples:
  atmux agent capture planner
  atmux agent capture atmux-myrepo-planner --lines 300
  atmux team capture platform

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabewillen/atmux](https://github.com/gabewillen/atmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
