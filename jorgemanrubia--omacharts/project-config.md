---
trigger: always_on
description: Anything the app can do, `omacharts` can do from a terminal, in the same
---

# Working on Omacharts

## Every feature ships with its command

Anything the app can do, `omacharts` can do from a terminal, in the same
change that adds it. A feature with no command is not finished.

This is not a nice-to-have that fell out of building a CLI. It is the point:
a capability you can only reach by clicking cannot be scripted, cannot be
tested from the outside, and cannot be used by an agent at all. The moment
one feature is exempt, the CLI stops being something anybody can rely on,
and the next person has no reason to keep it true either.

### What needs a command, and what does not

In scope — anything that changes **what you are looking at**, or **what is
stored as content or structure**:

- watchlists, their sections, and the symbols in them
- chartbooks: creating, renaming, deleting, switching
- chart layouts: splitting, closing, which chart holds what
- a chart's symbol, resolution, indicators, bar style, session, link group
- which watchlist a chartbook shows, and which link group a watchlist drives
- settings that are real preferences
- cache management

Out of scope — **presentational geometry**, which only means anything while a
window is on screen:

- sidebar width, split ratios, divider positions, indicator pane heights
- which pane is maximized
- scroll and zoom position

Those exist to be dragged with a mouse. A command to set one to 289 pixels is
noise in the help output that makes the useful commands harder to find.

The test to apply: **if a person would reasonably want to script it, or an
agent would need it to set up a working arrangement, it is in.** If it only
exists because something had to have a number, it is out. Note that splitting
a chart is firmly in and the *ratio* of the split is out — building a two-by-
two of particular symbols is worth scripting; nudging a divider to 47% is not.

## Commands act on what you are looking at

Every chart and chartbook verb defaults to the focused chart in the open
chartbook. That is what makes "add an RSI to this" work in one command rather
than a `status` call, a parse, and an identifier threaded into a second one —
and an agent that has to guess an identifier will eventually guess wrong.

Two rules keep that honest, and a new verb has to follow both:

- **Say what it acted on.** With an implicit target, naming the chart in the
  output is the only way anyone catches it reaching the wrong one. "added
  SMA(200) on pos:0 AAPL 1D in Macro", not "ok".
- **Refuse rather than fall back.** No window open means no focused chart.
  That is `EXIT_NO_WINDOW`, with its own message, never an answer taken from
  what was stored when the window last closed — an agent cannot tell stale
  from live and will act on it.

A chart is named back as `pos:N`, its position in the arrangement, not its id.
`materialise_book` hands out fresh pane ids every time it rebuilds, so an id is
only good until the next rebuild. Positions survive. Anything that reports a
chart, or remembers one across a rebuild, has to use the position.

## Where the surface is defined

`src/cli/spec.rs`, and nowhere else. One table describes every command, and
the parser, `--help`, the man page, the shell completions and the JSON surface
are all built from it. A description of a parser kept beside the parser is
wrong by the second release.

The authoritative machine-readable description is:

```
omacharts surface --json
```

That is what anything driving this from a script should read — every command,
every argument and flag with its type, the values enumerated ones accept, the
exit codes, and a worked example per command. `--help` is the path for people
and is held to the same standard, but prose cannot say that a flag takes an
integer without ambiguity, and the JSON can.

## The skill is part of the surface

`agents/skills/omacharts/SKILL.md` is the agent-facing skill, installed — only
ever on request — by `omacharts skill install`, which links it into whichever
agents the machine has. **One file for all of them:** Claude and Codex both read
a directory with a `SKILL.md` out of their own config, so a copy each would be a
second thing to drift. `agents/.claude-plugin/` wraps the same directory as a
Claude Code plugin, which only Claude has a use for; `src/cli/skill.rs` has the
reasoning, including why it is a symlink.

It exists for the two things this file and the surface cannot do: it is read
outside this repo, which is how an agent anywhere on the machine knows to reach
for omacharts at all, and it carries **workflows** rather than vocabulary,
because "a 2×2 of the majors at 15m with RSI on each,
linked" is several commands in an order plus a handful of facts no single
command's help contains.

It deliberately **does not** describe the command surface. That is generated,
it regenerates itself, and a prose copy of it is wrong by the second release —
confidently wrong, which is worse for an agent than nothing at all. One line
points at `omacharts surface --json` and that is the whole of it.

Which makes drift the only real way this gets worse, so it is tested:
**`every_command_in_the_agent_skill_is_a_command_that_runs`** pulls every
`omacharts` line out of the skill's fenced blocks and runs each block, in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorgemanrubia/omacharts](https://github.com/jorgemanrubia/omacharts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
