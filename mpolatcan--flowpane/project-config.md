---
trigger: always_on
description: A Claude Code plugin that replaces the default Workflow progress list with a
---

# flowpane

A Claude Code plugin that replaces the default Workflow progress list with a
live drawing of the run: a phase-by-phase graph, a stack, or a timeline in the
pane beside the transcript, with each agent's state, timing, token spend, tool
calls and prompt.

It draws into a terminal grid. Everything it knows comes from files a running
workflow writes, and everything it says has to fit in cells.

## Commands

```bash
claude plugin test .          # the test suite — 275 tests across 21 files
bunx tsc --noEmit             # typecheck (hooks/ and types/ only; see tsconfig.json)
```

Use `claude plugin test .`, not `bun test`. The tests import
`claude-code/testing`, which the plugin test runner supplies and a bare `bun
test` does not; bare `bun test` fails to resolve it.

The developer tools under `dev/` all take a run and draw it without a session:

```bash
bun dev/preview.ts [--animate|--journal <dir>|--settings|--menu theme|--about|--plain]
bun dev/shot.ts [--journal <dir>] [--port 8731]   # the pane in a browser, live, at any size
bun dev/stress.ts                                  # wide fan-outs and a narrow body
bun dev/lines.ts                                   # every line of every run at every size
bun dev/audit.ts                                   # every run, layout and size: nothing throws, every cell legible
bun dev/checkpic.ts <dir>                          # the bands cover the canvas, every node has a Button
bun dev/contrast.ts                                # every theme's roles against its ground, its edges against its states
bun dev/edges.ts <dir>                             # the graph derived for a run, and how it was derived
bun dev/recover.ts <sessionId> [home]              # what a session's runs rebuild to
bun dev/dryrun.ts <workflow>                       # a workflow's control flow, stubbed, no agents spent
bun dev/checkmeta.ts                               # the version the pane shows is the one the manifest ships
sh  dev/reload.sh                                 # install this tree as flowpane-dev, beside the released build
```

`dev/reload.sh` installs this tree *beside* the released plugin rather than in
place of it: `~/claude-dev-marketplaces/flowpane-dev` holds a copy of the repo
with the three names the engine keys off moved aside — the manifest's `name`,
`COMMAND` and `PANE_ID`. Renamed, both can be enabled at once: `/flowpane` stays
the released build and `/flowpane-dev` draws this tree. Unrenamed they collide on
every one of them — one command, one pane id, one `tool.call` hook opening one
pane. They are patched in the copy, never in the repo, so what ships is unchanged.

The title the pane draws is left alone. A build is told apart by the version it
already prints, and a product name bent out of shape to serve a dev install is a
dev install leaking into the picture.

A *copy*, not a symlink. A symlinked plugin source installs files that look
right — the engine's cache comes out byte-identical to the tree — and then never
loads, with no error printed anywhere: the command comes back `Unknown command`
while `claude plugin details` still reports the plugin as installed. So the
reload mirrors the tree in with `rsync`, then drops the engine's cached copy,
since an install over a version already installed is a no-op. Restart the session
afterwards: hooks are read once, when it starts.

Both builds watch the same journals and both open a pane on a Workflow launch,
so with the two enabled a live run draws twice. `/flowpane` toggles the released
one shut.

## Layout of the repo

| Path | What it holds |
| --- | --- |
| `hooks/register.ts` | The plugin itself: every hook, the journal tail, the run state |
| `hooks/journal.ts` | Reading a run off disk — journal, run file, agent transcripts |
| `hooks/shape.ts` | The run's shape: phases, passes, what fed what |
| `hooks/tree.ts` | The canvas as elements: coloured runs per row, and a Button over every hotspot |
| `hooks/layout.ts` | Where every band, card and column goes, at a given size |
| `hooks/paint.ts` | Everything drawn: cards, wires, header, dialogs, settings |
| `hooks/canvas.ts` | The grid: cells, colour mixing, line drawing, joints |
| `hooks/theme.ts` | Twelve themes, each ten colours, and the thirteen roles derived |
| `hooks/press.ts` | Keys and clicks |
| `hooks/about.ts` | What the pane says about itself: the dialog and `/flowpane about` |
| `tests/` | What the pane draws, read back off the canvas |
| `dev/` | The tools above |
| `docs/` | The design record: every decision, and what it replaced |
| `README.md` | What the plugin is, installing it, and the way in to `docs/` |

## Constraints the platform imposes

These are not preferences. Breaking one produces a pane that looks right in a
test and wrong in a terminal.

- **A `Button` carries no colour and no background.** `ButtonProps` has `key`,
  `label`, `hotkey`, `action`, `plain` and `hover` — no colour among them. A
  state mark, a rule or a fill drawn inside a Button's cells comes out in the
  label's own plain text. So every coloured cell stays outside every hotspot: the mark sits beside the
  name with a cell of air between them, and a card is framed in code points
  rather than filled.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mpolatcan/flowpane](https://github.com/mpolatcan/flowpane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
