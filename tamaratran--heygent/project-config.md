---
trigger: always_on
description: The complete suite needs the AppKit/Quartz and audio dependencies; without
---

# Working in this repo

## Verify

The complete suite needs the AppKit/Quartz and audio dependencies; without
them several modules cannot import and the failures are environmental, not
real:

```bash
uv run \
  --with 'aiohttp>=3.10,<4' \
  --with 'numpy>=1.26,<3' \
  --with 'sounddevice>=0.4.6,<1' \
  --with 'claude-agent-sdk>=0.2,<1' \
  --with pyobjc-framework-Cocoa \
  --with pyobjc-framework-Quartz \
  --with pyobjc-framework-ApplicationServices \
  python -m unittest discover -s tests -p "test_*.py"
```

One module: `python3 -m unittest tests.test_logging -v`.

The tmux + Claude Code smoke test uses a real Claude account, so it is
separate and not part of the suite:
`env -u ANTHROPIC_API_KEY python3 -u tests/smoke_interactive.py`.

The computer-use smoke test drives the real screen (TextEdit) and needs
the Accessibility and Screen Recording permissions, so it too is separate:
`python3 -u tests/smoke_computer.py`. The Quartz bindings it and the driver
need are not a manual install: `conduct.py` and `conductor/computer.py`
declare `pyobjc-framework-Quartz` and `-ApplicationServices` inline, and a
worker is handed the driver as `uv run --script conductor/computer.py`
(`computer.cli_command()`), so uv resolves them wherever the repo lives.

## Debugging a run

`~/.voice-conductor/logs/conductor-<pid>.jsonl` is the one place to look:
JSONL, one line per event, one file per process (two writers on one file
lose lines to a rotation race). Rotates at 10 MB keeping 5 backups; the 20
most recent files survive. Both launchers print their path and a `run_id`
at startup. Globbing `conductor-*.jsonl` reads every run at once.

Do not ask the user to reproduce the problem with extra printing added.
The log already has it, including tracebacks from failures that are caught
and swallowed. Start here:

Spell the path out in each command. With an unset `$LOG` the shell expands
it to nothing, `jq` reads stdin instead, and the command exits 0 with no
output - indistinguishable from a clean log.

```bash
# 1. what failed
jq -c 'select(.level=="error")|{event,message,task_id}' \
  ~/.voice-conductor/logs/conductor-*.jsonl

# 2. why - the real traceback, not a summary
jq -r 'select(.exception)|.exception.traceback' \
  ~/.voice-conductor/logs/conductor-*.jsonl | tail -30

# 3. the interaction it happened in (mic -> Manager -> worker)
TRACE=$(jq -r 'select(.trace_id!="")|.trace_id' \
  ~/.voice-conductor/logs/conductor-*.jsonl | tail -1)
jq -r --arg t "$TRACE" 'select(.trace_id==$t)
  |"\(.component)\t\(.event)\t\(.message[0:60])"' \
  ~/.voice-conductor/logs/conductor-*.jsonl

# 4. one worker's whole life, in file order (already chronological)
jq -r 'select(.task_id=="task_xxxx")
  |"\(.timestamp)\t\(.component)\t\(.event)\t\(.message[0:60])"' \
  ~/.voice-conductor/logs/conductor-*.jsonl
```

Reading the result:

- `task.approval_required` with no later `approval.resolved` for that task
  means a worker is blocked on a decision nobody made.
- `task.worker_gone` means the watchdog found a running task with no worker behind it and reclassified it as interrupted. `window=gone` is the PTY itself missing; `window=empty` is the window still listed but no process in it (a cmux workspace outlives the claude that exited inside it, and until this check that task stayed a busy worker for ever - one of the three slots create_task will fill).
- `task.failed` with `tmux session ended` is the per-session watcher giving
  up. It is preceded by `runtime.session_missing` (first missed poll) and
  followed by `runtime.watch_ended`; if `cmux.listing_failed` sits just
  before them, cmux stopped answering `workspace list` and the sessions
  were probably alive - check `pgrep -f 'claude --permission'` before
  believing it. `runtime.session_found` means the scare passed;
  `runtime.session_process_alive` means the host lost the session but its
  claude process is running and it is still being watched.
  `cmux.listing_stale` means cmux has not answered for 30 s and its last
  good listing no longer stands in for it.
- `boss.watcher_restarted` means the Boss was alive but nobody was reading
  its transcript (its replies were not being spoken); a turn or a worker
  update restarted the watcher, which delivers what was written meanwhile.
- `launch.unconfirmed` (with `cmux.launch_unconfirmed` carrying the pane
  text) means the host could not see the launch line taken; the launch
  is then judged by the session's transcript - a `task.failed` after it
  is real and the session was closed, a `runtime.session_created` means
  the worker started anyway. It is not, by itself, a worker that never
  started.
- `boss.words_queued` is an utterance typed into the Boss while it was
  mid-turn (`manager.turn_overlapping` is the same moment, conductor
  side). It sits in Claude Code's input box until the Boss looks up; a
  `boss.turn` with `folded: true` is one that was read into the earlier
  turn and answered there. `boss.words_lost` is one Claude Code never
  wrote to its transcript (a later one was read instead): it gets no
  answer, and the timeline says "the Boss never read: ...". Which
  utterance a reply answers is matched by the words of the user line,
  never by order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tamaratran/heygent](https://github.com/tamaratran/heygent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
