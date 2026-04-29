---
trigger: always_on
description: AI agent monitor for your terminal. Like btop++, but for AI coding agents.
---

# abtop

AI agent monitor for your terminal. Like btop++, but for AI coding agents.

Supports Claude Code and Codex CLI.

## Architecture

```
src/
├── main.rs                 # Entry, terminal setup, event loop, --setup flag
├── app.rs                  # App state, tick logic, key handling, summary generation
├── setup.rs                # StatusLine hook installation (abtop --setup)
├── ui/
│   └── mod.rs              # All panels in single file: header, context, quota,
│                           # tokens, projects, ports, sessions, footer
├── collector/
│   ├── mod.rs              # MultiCollector orchestration, orphan port detection
│   ├── claude.rs           # Claude Code: session discovery, transcript parsing
│   ├── codex.rs            # Codex CLI: session discovery via ps+lsof, JSONL parsing
│   ├── process.rs          # Child process tree (ps) + open ports (lsof) + git stats
│   └── rate_limit.rs       # Rate limit file reading (~/.claude/abtop-rate-limits.json)
└── model/
    ├── mod.rs              # Re-exports
    └── session.rs          # AgentSession, SessionStatus, RateLimitInfo,
                            # ChildProcess, OrphanPort, SubAgent
```

## Layout

```
┌─ ¹context (token rate sparkline + per-session context bars) ─────────┐
│  ▁▃▅▇█▇▅▃▁▃▅▇██                       S1 abtop       ████████ 82%  │
│  token rate (200pt history)            S2 prediction  █████████91%⚠ │
│                                        S3 api-server  ███      22%  │
└──────────────────────────────────────────────────────────────────────┘
┌─ ²quota ─────┐┌─ ³tokens ───┐┌─ projects ───┐┌─ ⁴ports ──────────┐
│ CLAUDE       ││ Total  1.2M ││ abtop        ││ PORT  SESSION  CMD │
│ 5h ████ 35%  ││ Input  402k ││  main +3 ~18 ││ :3000 api-srv node│
│   resets 2h  ││ Output  89k ││              ││ :8080 predict crgo│
│ 7d ██ 12%    ││ Cache  710k ││ prediction   ││                    │
│              ││ ▁▃▅▇█▇▅▃▁▃▅││  feat/x +1~2 ││ ORPHAN PORTS       │
│ CODEX        ││ Turns: 48   ││              ││ :4000 old-prj node│
│ 5h █ 9%     ││ Avg: 25k/t  ││ api-server   ││                    │
│ 7d ██ 14%    ││             ││  main ✓clean ││                    │
└──────────────┘└─────────────┘└──────────────┘└────────────────────┘
┌─ ⁵sessions ─────────────────────────────────────────────────────────┐
│ ►*CC 7336 abtop  ● Work opus  82% 1.2M  48  Edit src/pay.rs       │
│  >CD 8840 pred   ◌ Wait sonn  91% 340k  12  waiting                │
│ ─────────────────────────────────────────────────────────────────── │
│  SESSION 7336 · /Users/graykode/abtop                               │
│  Stripe payment integration...                                      │
│  └─ Edit src/pay.rs                                                 │
│  CHILDREN: 7401 cargo build                                         │
│  SUBAGENTS: explore-data ✓12k · run-tests ●8k                      │
│  MEM 4f · 12/200 │ v2.1.86 · 47m                                   │
└──────────────────────────────────────────────────────────────────────┘
```

Panel rendering priority (top to bottom):
1. **Sessions** — always visible, gets priority allocation (min 5 rows, ideal = 2/session + 7)
2. **Mid-tier** (quota, tokens, projects, ports) — split equally, shown if space allows
3. **Context** — only renders when sessions have ideal height AND surplus >= 5 rows
4. **Header** (1 row) + **Footer** (1 row) — always present

Panel descriptions:
- **¹context**: Left = token rate braille sparkline (200-point history). Right = per-session context % bars with yellow/red warning.
- **²quota**: Claude + Codex rate limit gauges side-by-side (5h and 7d windows with reset countdown).
- **³tokens**: Total token breakdown (in/out/cache) + per-turn sparkline for selected session.
- **projects** (always visible): Per-project git branch + added/modified file counts.
- **⁴ports**: Agent-spawned open ports + orphan ports (from dead sessions). Conflict detection.
- **⁵sessions**: Full-width panel below mid row. Session list table (top) + selected session detail (bottom), separated by divider.

## Data Sources

All read-only from local filesystem + `ps` + `lsof`. No API calls, no auth.

### 1. Claude Code session discovery: `~/.claude/sessions/{PID}.json`
```json
{ "pid": 7336, "sessionId": "2f029acc-...", "cwd": "/Users/graykode/abtop", "startedAt": 1774715116826, "kind": "interactive", "entrypoint": "cli" }
```
- ~170 bytes. Created on start, deleted on exit.
- Scan all files, verify PID alive with `ps -p {pid} -o command=` containing `/claude`.
- Skip `--print` sessions (abtop's own LLM calls for summary generation).

### 2. Claude Code transcript: `~/.claude/projects/{encoded-path}/{sessionId}.jsonl`
Path encoding: `/Users/foo/bar` → `-Users-foo-bar`

Key line types:

**`assistant`** (tokens, model, tools):
```json
{
  "type": "assistant",
  "timestamp": "2026-03-28T15:25:55.123Z",
  "message": {
    "model": "claude-opus-4-6",
    "stop_reason": "end_turn",
    "usage": {
      "input_tokens": 2,
      "output_tokens": 5,
      "cache_read_input_tokens": 11313,
      "cache_creation_input_tokens": 4350
    },
    "content": [
      { "type": "text", "text": "..." },
      { "type": "tool_use", "name": "Edit", "input": { "file_path": "src/main.rs", ... } }
    ]
  }
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [graykode/abtop](https://github.com/graykode/abtop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
