---
trigger: always_on
description: Connect coding agents and other AI integrations to Aven.
---


Aven gives AI agents access to the same local task system you use in the TUI.
Coding agents can inspect project context, find ready or blocked work, create and
edit tasks, manage status, schedules, and relationships, and leave durable
handoff notes. Chat integrations can turn messages or transcripts into tasks
through the CLI and sync them to your other devices.

## Install the aven skill

```sh
aven skill install
```

`aven skill install` writes the bundled skill to every detected coding-agent skill directory. Detection checks Claude Code, OpenCode, Codex, and Pi user directories, plus matching agent config directories in the current workspace.

Use `--agent` to choose explicit targets:

```sh
aven skill install --agent claude
aven skill install --agent opencode
aven skill install --agent codex
aven skill install --agent pi
```

The user skill locations are:

- Claude Code: `~/.claude/skills/aven/SKILL.md`
- OpenCode: `~/.config/opencode/skills/aven/SKILL.md`
- Codex: `~/.codex/skills/aven/SKILL.md`
- Pi: `~/.pi/agent/skills/aven/SKILL.md`

Explicit targets are installed even when the agent directory is absent. The command reports a clear error when no supported agent is detected and no target is provided.

## Set up automatic priming

Run `aven prime` automatically when an agent session starts so its output is
available to the agent from the first model turn.

### Claude Code

Add a `SessionStart` hook to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "aven prime"
          }
        ]
      }
    ]
  }
}
```

### Pi

[Pi](https://pi.dev) extensions in `~/.pi/agent/extensions/` apply globally.
Create the directory, then add `~/.pi/agent/extensions/aven-prime.ts`:

```sh
mkdir -p ~/.pi/agent/extensions
```

```typescript
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  let primeOutput: string | undefined;
  let delivered = false;

  pi.on("session_start", async (event, ctx) => {
    if (event.reason === "reload") return;

    primeOutput = undefined;
    delivered = false;

    const result = await pi.exec("aven", ["prime"], {
      cwd: ctx.cwd,
      timeout: 30_000,
    });

    if (result.code !== 0) {
      ctx.ui.notify(
        `aven prime failed: ${result.stderr.trim() || `exit code ${result.code}`}`,
        "error",
      );
      return;
    }

    primeOutput = result.stdout.trim();
  });

  pi.on("before_agent_start", async () => {
    if (delivered || !primeOutput) return;

    delivered = true;

    return {
      message: {
        customType: "aven-prime",
        content: primeOutput,
        display: false,
      },
    };
  });
}
```

The extension uses two lifecycle events because they serve different purposes.
`session_start` runs `aven prime` once in the session working directory and
caches its output. `before_agent_start` is the point where Pi accepts context
for a model turn, so it injects the cached output as a hidden message on the
first turn only. Reload events are skipped to avoid injecting a second copy into
the same session.

Pi profiles can share the same extension source. For a profile rooted at
`~/.pi-epic`, link its extension directory to the default profile:

```sh
mkdir -p ~/.pi-epic/agent
ln -s ~/.pi/agent/extensions ~/.pi-epic/agent/extensions
```

Both profiles then discover `aven-prime.ts` through their global extension
directory. Run `/reload` in an open Pi session after editing the extension.

Other agent environments can use the same pattern: run `aven prime` at session
start and include its output in the agent context.

## Project context

aven infers the active workspace and project from the current directory. Start an agent from a repository directory and automatic priming loads the matching project context.

Use [Configuration](/configuration/) when directory names, workspace routes, or project path mappings need to be explicit. Use `aven doctor` from the same directory to inspect the active database, workspace, project, and routing decisions.

## Work on tasks with coding agents

A typical workflow looks like this:

1. Capture or triage work in aven.
2. Start your agent from the repository directory.
3. Let the startup hook load aven context.
4. Ask the agent to work on a specific ref, or to choose ready work.
5. Review the code change and the task note the agent leaves behind.

Example prompts:

```txt
Work on APP-7KQ9. Use aven for status and handoff notes.
```

```txt
Pick a ready docs task and complete it.
```

## Durable handoff

Descriptions hold the main task context: problem statement, scope, acceptance criteria, and links.

Notes hold durable handoff context: implementation decisions, blockers, partial progress, and review findings. Agent notes survive chat sessions, branch switches, worktrees, and machine restarts.

:::caution[Protect durable context]
Keep secrets out of titles, descriptions, labels, projects, notes, and logs.
:::

## Skill or prime?

Use the installed skill and priming for different levels of commitment:

| Setup | What the agent receives | Choose it when |
| --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raine/aven](https://github.com/raine/aven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
