---
trigger: always_on
description: algotutor hosts independent training tracks ("courses"). Each course has its own concepts,
---

# Algotutor — Multi-Course Training Project

algotutor hosts independent training tracks ("courses"). Each course has its own concepts,
progress, problems, cards, mistakes, re-solve schedule, and retention. The user is enrolled
in one or more courses and trains in **one course at a time**.

## Active Course Resolution

**Always run this before any other flow.**

1. Read `state.json` at the repo root. It looks like:
   ```json
   {
     "enrolled": ["algos", "conc"],
     "active": "algos",
     "default": "algos",
     "default_agent": null
   }
   ```
2. The course slug at `state.active` is the **active course**. Throughout this document,
   `<active>` is a placeholder that means *that slug*. So `courses/<active>/progress.md`
   resolves to `courses/algos/progress.md` when active is `algos`.
3. If `state.json` is missing, ask the user to run `make init`. Do not invent state.
4. If the user types `train <course>`, `review <course>`, `mistakes <course>`, or
   `reset <course>` with a course slug, **first set `state.active` to that slug** (the slug
   must be in `state.enrolled`), then continue. Persist the change to `state.json`.
5. If the user types `train` with no argument and `state.enrolled` has more than one course,
   resolve to `state.active` (last-used), but tell them in one line which course you're
   training: "Training `<active>` (say `train conc` to switch)." If they have only one
   enrolled course, train that one silently.
6. **Course labels in prose.** When referring to the active course in any user-facing
   message, use the human-readable **Label**, never the raw slug. The label mapping is
   defined in `internal/courses/courses.go`:
   - `algos` → **Algos**
   - `conc` → **Concurrency**
   Use the slug only in file paths, `state.json` values, and command examples (e.g.
   `train conc`). In sentences, write "the Concurrency course" or "Training Concurrency",
   not "the conc course" or "Active course is `conc`".

## Project Structure

```
state.json                       — active course, enrolled list, default agent
main.go                          — current problem template (active course)
main_test.go                     — present only when active course is `conc`
docs/                            — project-wide + shared mechanics docs
  agents.md                      — per-agent setup
  cards.md                       — spaced-repetition card format (shared)
  mix.md                         — mix-mode mechanics (shared)
  resolve.md                     — re-solve mechanics (shared)
courses/<slug>/                  — per-course universe
  progress.md                    — concept levels for this course (user state)
  progress.template.md           — blank progress (zeros)
  current.md                     — current problem pointer
  problems/                      — NNN.md per problem
  problem-bank.md                — curated problems by concept and level
  cards.json                     — review cards (auto-created)
  mistakes.json                  — mistake log (auto-created)
  resolve.json                   — re-solve schedule (auto-created)
  retention.json                 — per-concept retention (auto-created)
  mix.json                       — mix-session state (auto-created)
  docs/
    concepts.md                  — concept list with prerequisites (course-specific)
    go-gotchas.md                — language traps relevant to this course
    mistakes.md                  — course-specific mistake taxonomy + drill rules
cmd/
  init/                          — huh-driven onboarding (`make init`)
  start/                         — flips active course + launches agent (`make train`)
  review/                        — review TUI (`make review [course]`)
internal/
  courses/                       — state.json read/write, path helpers
  migrate/                       — one-shot legacy → multi-course migration
  cards/                         — FSRS card storage
  review/                        — review TUI Bubble Tea model
```

When this document references a path like `courses/<active>/progress.md`, substitute
`<active>` with the active course slug from `state.json`.

## Working Files Per Course

The user works on `main.go` (and `main_test.go` for concurrency) at the **repo root**, not
inside the course directory. The agent always edits the root files, but reads the problem
statement from `courses/<active>/problems/NNN.md`.

| Active course | Files at root              | Validation (under the hood)   |
|---------------|----------------------------|-------------------------------|
| `algos`       | `main.go`                  | `go run .` + `fmt.Println`    |
| `conc`        | `main.go` + `main_test.go` | `go test -race .`             |

The user validates with **`make run`** — a single dispatcher that picks the right command
based on the active course. Tell the user "run `make run` to sanity-check before
`check`," not the underlying Go command. The agent's `check` flow is the *evaluation*
(grade, log mistakes, advance levels); `make run` is the *local smoke test* (does it
compile / pass tests).

When the active course changes, swap the contents of these files to match the new course's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zuzuleinen/algotutor](https://github.com/zuzuleinen/algotutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
