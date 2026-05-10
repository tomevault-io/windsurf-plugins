---
trigger: always_on
description: > **Keep this file under 200 lines.** Commands only. Project conventions → `rules/project-context.md`. Everything the agent needs to remember → `.claude/memory/`. Long sessions → `/learn` then `/compact`.
---

# [Project Name] — Claude Code + Codex Project Context

> **Keep this file under 200 lines.** Commands only. Project conventions → `rules/project-context.md`. Everything the agent needs to remember → `.claude/memory/`. Long sessions → `/learn` then `/compact`.

---

## Session Commands

| Tier | Commands |
|------|----------|
| **Core** | `Start Session` · `End Session` |
| **On Demand** | `Plan` · `Debug Session` · `/learn` · `/evolve` · `Check Drift` · `Guard Check` · `Pre-Ship Check` · `Code Health` · `Mode` · `Estimate` · `Handoff` · `Search Memory` · `Generate Guards` · `Generate Skills` · `Update Kit` |
| **Opt-In** | `Team Pull` / `Team Push` · `Sync Memory` / `Pull Memory` |

---

### `Setup Memory`
If `setup.py` exists run it. Otherwise: `python -c "import urllib.request; exec(urllib.request.urlopen('https://raw.githubusercontent.com/YehudaFrankel/clankbrain/main/install.py').read().decode())"`

### `Start Session`
1. Run `python tools/memory.py --session-start`
2. Read `STATUS.md` → `memory/lessons.md` → `memory/decisions.md` → `memory/tasks/regret.md` → `memory/tasks/todo.md`
3. Scan `memory/plans/` — surface any file with `Status: Draft` or `On Hold`
4. Report: "Session N ready. Last change: [X]. What are we working on?"

### `End Session`
1. Run `/learn` — extract lessons + decisions from this session
2. Update memory files for everything changed (see Auto-Save Rule)
3. Update `STATUS.md` — increment session, one-line summary
4. Run `python tools/memory.py --check-drift`
5. If on a team: share what you learned → `Team Push`
6. Report: "Session N complete. Updated: [list]. Memory clean."

### `Plan [feature]`
Invoke the `plan` skill. Opens/creates `memory/plans/[slug].md`, walks problem → research → options → decision → spec. Always show the **full plan file** after every update — never a diff or summary.

### `Debug Session`
Invoke the `debug-session` skill: reproduce → isolate → hypothesize → fix only the confirmed root cause → verify → log to `tasks/errors.md`.

### On-Demand Commands
| Command | Action |
|---------|--------|
| `Check Drift` | `python tools/memory.py --check-drift` — fix any found |
| `Guard Check` | `python tools/memory.py --guard-check` — PASS/FAIL per guard |
| `Pre-Ship Check` | guard-check + drift-check + session edit count + `git diff --stat` |
| `Code Health` | Scan for console.log, hardcoded values, missing error handling, dead code, files >500 lines |
| `Mode [develop\|review\|safe\|deploy]` | Set + enforce tool access constraints for the session |
| `Estimate: [task]` | Read files → complexity + risk + file list → write to todo.md if confirmed |
| `Handoff` | Generate `HANDOFF.md` from STATUS + todo + decisions + errors + gotchas |
| `Search Memory: [topic]` | `python tools/memory.py --search "[topic]"` |
| `Progress Report` | `python tools/memory.py --progress-report` |
| `Kit Health` | `python tools/memory.py --kit-health` — fix FAILs immediately |
| `Generate Guards` | Invoke `generate-guards` skill |
| `Generate Skills` | Invoke `generate-guards` then scan stack for useful project skills |
| `Analyze Codebase` | Scan all JS/CSS/backend files, update memory files with findings |
| `Install Memory` | Scan codebase, fill memory files, copy to system memory path |
| `Update Kit` | Run `update.py` if present, otherwise fetch from clankbrain repo |
| `/learn` | Invoke `learn` skill — extract lessons, decisions, rejected approaches |
| `/evolve` | Invoke `evolve` skill — patch failing skills, cluster patterns (every 3–5 sessions) |

### Team (opt-in)
| Command | What it does |
|---------|-------------|
| `Setup Team [url]` | You're the manager. Run once. Sends teammates the URL to join. |
| `Join Team [url]` | You're a new member. Run once with the URL your manager sent. |
| `Team Push` | Share what you learned with the team. Run at End Session. |
| `Team Status` | Check last sync times and what's been shared. |

Team Pull runs automatically at Start Session — no command needed.

### Sync (opt-in)
- `Sync Memory` / `Pull Memory` → `python sync.py push` / `python sync.py pull`
- Setup: `python sync.py setup [repo-url]`

---

## Skill Map

| Workflow | Skills in Order |
|----------|----------------|
| **Build to Learn (discovery)** | Declare `build-mode: learn` → `product-risk` → `prototype-hypothesis` → *(prototype)* → `parallel-prototypes` (if multiple options) → log to `prototype_log.md` |
| **Build to Earn (delivery)** | Declare `build-mode: earn` → `search-first` → `plan` → *(code)* → `code-reviewer` → `verification-loop` → `/learn` |
| **Bug Fix** | `debug-session` → *(fix)* → `verification-loop` → `/learn` |
| **End of Session** | `/learn` → `/evolve-check` → `/evolve` *(every 3–5 sessions)* |
| **Maintenance** | `check-drift` → `guard-check` → `code-health` |
| **Memory** | `/recall [topic]` · `search-memory` · `/forget [topic]` |

Full agent orchestrations (human-in-the-loop breakpoints): see `.claude/agents/`

---

## Auto-Save Rule

After any code change, immediately update the relevant memory file — don't wait for End Session:

| What changed | Update this |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YehudaFrankel/clankbrain](https://github.com/YehudaFrankel/clankbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
