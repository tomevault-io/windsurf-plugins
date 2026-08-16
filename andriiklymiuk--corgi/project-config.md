---
trigger: always_on
description: Agent mode makes your machine an always-on, multi-repo host for Claude Code's
---

# Agent mode

Agent mode makes your machine an always-on, multi-repo host for Claude Code's
[Remote Control](https://code.claude.com/docs/en/remote-control).

It is opt-in. If you never run `corgi agent init`, nothing changes.

## What it is for

Remote Control already gives you a Claude Code session on your own machine,
driven from the Claude app or claude.ai/code. Two things stop it being the thing
you actually want:

1. **It is not always on.** Its docs are explicit: *"If you close the terminal,
   quit VS Code, or otherwise stop the `claude` process, the session goes
   offline"*, and *"if your machine is awake but unable to reach the network for
   more than roughly 10 minutes, the session times out and the process exits"*.
   So you have to remember to arm it — and forgetting is the failure this exists
   to remove.
2. **It sees one directory.** A corgi stack is several repositories, databases,
   and the env wiring between them. Remote Control sees none of that.

corgi fixes exactly those two. It does not reimplement sessions, streaming,
approvals, or cost tracking — Remote Control does all of that well, and doing it
again would be worse.

```
  phone / claude.ai/code
        │
        ▼
  claude rc --spawn=worktree      ◄── supervised by corgi, not replaced
        │                                   │
        │ calls MCP tools                   │ restarts after the 10-minute exit,
        ▼                                   │ a crash, or a reboot; holds a wake lock
  corgi mcp                            corgi agent serve
        ├─► which stack is "the recipe app"?
        ├─► a worktree per repo, one branch
        └─► one diff across every repo
```

## Quick start

```bash
cd ~/dev/your-stack
corgi agent init                 # register AND enable this stack
corgi agent install              # start at login (launchd / systemd)
corgi agent status               # what is running, and under which account
```

`corgi agent scan <dir>` registers stacks it finds but **does not enable them**.
Supervision is opt-in per workspace: scanning a projects folder should not
quietly spawn a Claude session for every stack in it. Run `corgi agent init` in
the ones you actually want running.

Then open the Claude app. Nothing to arm.

To check it can work before committing to it:

```bash
corgi agent doctor
corgi agent serve --foreground   # run it in this terminal and watch
```

## Commands

| command | what it does |
|---|---|
| `corgi agent init` | register this stack, write `.corgi/agent.yml` |
| `corgi agent scan <dir>` | find stacks under a directory and register them (does not enable) |
| `corgi agent serve` | supervise Remote Control for every enabled workspace |
| `corgi agent install` / `uninstall` | start (or stop starting) at login |
| `corgi agent status [--json]` | what is running, restarts, which account |
| `corgi agent doctor [--json]` | can this work here, and what to fix |
| `corgi agent workspaces` | list, `forget`, `relocate` |
| `corgi agent resolve <name>` | what "the recipe app" resolves to |
| `corgi agent brief [id]` | what the last session was working on before it restarted |
| `corgi agent stop` | stop the daemon |

## Restarts, and being told about them

Remote Control exits on the ten-minute network timeout, on a crash, and on
reboot. corgi restarts it with capped backoff and **says so**:

> `rc restarted 14:32 · previous session ended (network timeout) · worktrees kept`

The notification matters. A relaunched Remote Control starts a **new** session,
so the previous conversation's context is gone. Restarting silently would look
like continuity and cost you an hour of confusion.

### The handover brief

corgi cannot restore the conversation. What it can keep is the half that
survives on disk, captured in the gap between the old process exiting and the
new one starting — the only moment that state is both final and current:

```bash
$ corgi agent brief acme-stack
acme-stack
  ended   2026-08-14 14:32 (network-timeout)
  reason  remote control restarted — the previous session ended (network timeout)
  state   was on feature/referral · 1 repo has uncommitted changes
    api              feature/referral (worktree)
    web              feature/referral · uncommitted changes (worktree)
```

The worktrees are the part worth having. A branch spread across four
repositories is invisible from a fresh session's working directory, and nothing
else would tell the new session it exists. The summary line is appended to the
restart notification, so the lock screen says *where* as well as *that*.

Uncommitted work counts untracked files, unlike the check that guards worktree
removal. Creating files is the most common thing an agent does, and a note
calling that "clean" would be worse than no note. `.gitignore` is respected, so
build output does not inflate the count.

A session that ends because you asked it to gets no brief — you do not need a
handover note for something you just closed. Only the most recent is kept per
workspace, and `corgi agent workspaces forget` drops it, so a reused id cannot
surface another stack's branches.

From a phone, the same thing is `corgi_session_brief`. Call it first when
picking work back up.

Not every exit is worth retrying:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Andriiklymiuk/corgi](https://github.com/Andriiklymiuk/corgi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
