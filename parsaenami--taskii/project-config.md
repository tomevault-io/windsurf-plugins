---
trigger: always_on
description: Decision log for the `terminal-dashboard` project. Updated after every step/change.
---

# AGENTS.md

Decision log for the `terminal-dashboard` project. Updated after every step/change.

## Project summary

Full-screen terminal task manager / dashboard. Sections:
1. **Today** — today's tasks & appointments, mark done/undone, delete.
2. **Overdue** — previous days' undone tasks, shown in a distinct (warning) color.
3. **Reports** — progress for today/week/month + extra charts (7-day bar chart, streak).

Data persists to a JSON file in the project directory (`data/tasks.json`), loaded on every launch.

## Tech stack decisions

- **Language**: Go (chosen by user over Rust/ratatui, Python/Textual, Node/Ink).
- **TUI framework**: [Bubble Tea](https://github.com/charmbracelet/bubbletea) (Elm-architecture TUI).
- **Styling**: [Lip Gloss](https://github.com/charmbracelet/lipgloss) for layout/colors.
- **Components**: [Bubbles](https://github.com/charmbracelet/bubbles) (list, textinput) where useful.
- Why: modern, actively maintained, produces visually polished dashboards, compiles to a single static binary — easy to run with `go run .` or build.

## Effort-tier workflow (per user instruction)

- **Project structure & planning** → done at high effort, directly in the orchestrating session.
- **Feature development/implementation** → delegated to medium-effort subagents.
- **Low-level tasks** (file read/write helpers, lint, type/build checks, doc touch-ups, bug fixes) → delegated to Haiku 4.5 high-effort subagents.
- This file is updated after every step/change with what was done and why.

## Project structure (as scaffolded)

```
terminal-dashboard/
├── AGENTS.md
├── go.mod
├── main.go
├── internal/
│   ├── model/       # Task struct + JSON store (load/save)
│   ├── ui/          # Bubble Tea root model, panes, theme, keybindings
│   └── stats/       # completion % / streak calculations for reports
├── data/
│   └── tasks.json   # runtime data file (gitignored)
└── .gitignore
```

## Data model (planned)

```go
type Task struct {
    ID        string
    Title     string
    Done      bool
    Date      string // YYYY-MM-DD, the day the task belongs to
    Time      string // optional HH:MM for appointments, "" if none
    CreatedAt time.Time
    DoneAt    *time.Time
}
```

Store: whole-file JSON array, saved synchronously after every mutation (add/toggle/delete). Simple and safe at this scale (personal single-user todo list).

## Layout (planned)

Two-column full-screen layout:
- Left column, top: **Today** pane (add/toggle/delete).
- Left column, bottom: **Overdue** pane (read-mostly: toggle/delete allowed, no add — tasks are inherited from prior days), rendered in a warning/orange-red color.
- Right column: **Reports** pane — today/week/month completion progress bars, 7-day completion bar chart, current streak.

Keybindings: `a` add, `space`/`enter` toggle done, `d` delete, `↑/↓` or `j/k` navigate, `tab` switch focus between Today/Overdue, `q`/`ctrl+c` quit.

## Implementation details (as built)

- **Streak definition**: walking backwards from today, a day counts toward the streak only if it has ≥1 task AND all of that day's tasks are Done. A day with zero tasks breaks the streak. Today itself doesn't break an already-earned streak while still in progress (incomplete or empty) — it just isn't counted until finished.
- **Week boundary**: rolling last 7 days (today + 6 prior), not calendar Mon–Sun. Chosen so "This Week" progress and the 7-day bar chart describe the same window.
- **Month boundary**: calendar month (1st of current month through today).
- **Add-task flow**: single inline text input (`a` to open, Enter confirms, Esc cancels). Typing a trailing `HH:MM` token (e.g. `Buy milk 14:30`) is auto-detected as the appointment time and stripped from the title — no separate time prompt.
- Dependencies: `charmbracelet/bubbletea` v1.3.10, `charmbracelet/lipgloss` v1.1.0, `charmbracelet/bubbles` v1.0.0.
- Theme: Tokyo Night–inspired dark palette, rounded borders, focus-aware border colors, overdue tasks rendered in a warning/amber-red color distinct from today's tasks.
- Known minor inconsistency: `internal/ui/keys.go` defines a `key.Binding` keymap, but `app.go` currently switches on raw `msg.String()` instead of `key.Matches`. Functionally equivalent, just not wired through the abstraction — candidate cleanup for a future pass.
- `go build ./...` and `go vet ./...` both clean; `gofmt` applied.

## Change log

- **2026-08-19** — Initialized git repo, `go mod init terminal-dashboard`, created directory skeleton (`internal/model`, `internal/ui`, `internal/stats`, `data`), `.gitignore`, and this AGENTS.md. Stack decision (Go + Bubble Tea) made via user choice among 4 presented options.
- **2026-08-19** — Implemented full app via medium-effort subagent: `main.go`, `internal/model/{task,store}.go`, `internal/stats/stats.go`, `internal/ui/{theme,keys,today,overdue,reports,app}.go`. Today/Overdue/Reports panes, JSON persistence to `data/tasks.json`, full keybinding set (a/space/enter/d/tab/j-k/arrows/q). Build and vet clean.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parsaenami/taskii](https://github.com/parsaenami/taskii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
