---
trigger: always_on
description: - Never add `Co-Authored-By: Claude...` or `Claude-Session:...` trailers to git commit messages.
---

## Git Commit Guidelines
- Never add `Co-Authored-By: Claude...` or `Claude-Session:...` trailers to git commit messages.


# Limoni — working notes for Claude Code

Limoni is a terminal UI engine for Go: a flat 1D cell grid, a double-buffered ANSI
diff, and zero heap allocations on the render hot path. It also ships a software
3D rasteriser, four image protocols, charts, markdown, and an accessibility tree —
none of which the other Go TUI libraries have.

Longer architectural background lives in `.agents/skills/limoni_development/skill.md`
and `docs/`. This file is the short version plus the things that are easy to get
wrong.

Four skills in `.claude/skills/` carry the detail for the areas that bite
hardest, and load themselves when the work touches them:

- `limoni-agent-surface` — the semantic tree, the automation socket,
  `cmd/limoni-mcp` and `uitest`: invariants, the bugs they came from, and how to
  test an agent surface (unit, mutation, PTY, a real agent).
- `limoni-text-rendering` — grapheme clusters, widths, mode 2027, the cluster
  table, and the tools widgets use to measure and cut text.
- `limoni-performance` — where allocations hide in a frame, how to find them
  with a profile, how to compare benchmarks, and the release steps.
- `limoni-zest` — the log viewer: its store, filtering and LogView, and the
  four places it is verified.

---

## Verify before you claim anything works

```bash
go build ./...
go vet ./...
go test ./...
go test -race . ./component ./core/engine ./core/terminal ./testkit ./widgets ./layout ./core/accessibility ./core/driver

# Allocation budgets — these are enforced in CI and must stay at 0
go test ./core/buffer -run '^$' -bench 'BenchmarkDiff_' -benchmem
go test ./widgets    -run '^$' -bench . -benchmem
```

The WebAssembly build is part of the contract, not an afterthought:

```bash
GOOS=js GOARCH=wasm go build -o /tmp/limoni.wasm ./examples/wasm
```

---

## Non-negotiables

**Zero allocations on the hot path.** `Draw` implementations must not allocate.
Write a `-benchmem` benchmark for every new widget and check `0 B/op`. If a widget
genuinely cannot avoid allocating, say so in its doc comment with the measured
number rather than leaving it for someone to discover.

**Comments in English.** Parts of the codebase still carry Turkish comments
(`widgets/`, `core/terminal/`, `graphics/`, `layout/`, `animation/`). New code is
English; converting the rest is welcome. User-facing docs stay bilingual —
`README.md` / `README_TR.md` and `docs/` / `docs/tr/` — that is deliberate.

**Benchmark honesty is a hard rule.** This repository publishes comparisons against
Bubble Tea and Ratatui. Read `docs/benchmark-methodology.md` before touching any
number, and follow it:

- Every comparison carries an explicit version label. "Bubble Tea" alone is not a
  claim, "Bubble Tea v1.3.10" is.
- If the runners measure different pipelines, say so rather than quoting the ratio.
  The v1 Bubble Tea runner measures view-string construction only; the Limoni
  runner measures drawing *plus* the full ANSI diff. That gap is documented, not
  hidden.
- Workloads with no equivalent on the other side (`empty-frame`, `mouse-hit-test`,
  `async-update-burst`) are reported without ratios.
- A number that looks too good is a bug in the harness until proven otherwise. One
  earlier run reported a 4,700× advantage; it was entirely a harness artifact.

**Compiling is not verifying.** Two real bugs shipped in this repo because someone
checked that the code built and stopped there: the WebAssembly demo called
`Program.Run` (which only drives the message loop) instead of `RunTerminal`, so it
rendered nothing at all; and the browser ran in 16 colors because capability
detection reads environment variables that do not exist under `js/wasm`. Both were
found by running the thing, not by building it.

---

## Two application models, both first-class

```go
// Immediate mode — dashboards, 3D viewers, games, animation
limoni.Run(func(f *limoni.Frame, ev *limoni.Event) bool { ... })

// Declarative (Elm architecture) — forms, wizards, CRUD, async work
limoni.RunProgram(ctx, &model{})          // batteries included
limoni.NewProgram(&model{}, opts...)      // when you own the terminal
```

`Model`, `Msg`, `Cmd`, `UpdateResult` and the runtime message types are re-exported
from the root package; the runtime itself lives in `core/engine`. Options that
collide with the immediate-mode `AppOption` names are spelled
`WithProgramFPS` / `WithProgramCatchCtrlC` / `WithoutProgramQuitKeys`.

Both modes are context-aware: `limoni.RunWithContext(ctx, fn)` and
`limoni.NewApp(term, opts...).Run(ctx, fn)` for immediate mode. An `App` owns
its wakeup channel, so several run in one process (one per SSH session in
`examples/ssh_server`); package-level `limoni.Wakeup()` wakes all of them.

Note: `.agents/skills/limoni_development/skill.md` still refers to `runtime.New`.
The package was renamed to `core/engine`; that doc is stale in places.

---

## Package map

| Path | What lives there |
| :--- | :--- |
| `core/cell` | `Cell`, `Style`, `Rect`, `Context`, rune widths |
| `core/buffer` | Flat 1D buffer, the ANSI diff engine, snapshots |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thebanri/limoni](https://github.com/thebanri/limoni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
