---
trigger: always_on
description: AGENTS.md is the single source of truth for this repo's agent guidance.
---

<!--
  AGENTS.md is the single source of truth for this repo's agent guidance.
  CLAUDE.md and GEMINI.md at the repo root are symlinks to this file.
  Edit AGENTS.md only — the other two follow.
-->

> [!IMPORTANT]
> **Single source.** `CLAUDE.md` and `GEMINI.md` at the repo root are symlinks
> to this file. **Edit `AGENTS.md` only** — the other two follow.
>
> AGENTS.md is canonical because it's the cross-vendor spec
> ([agents.md](https://agents.md/)) read natively by Codex, Cursor, Jules,
> Gemini CLI, Windsurf, Aider, Zed, and ~20 other agents. Anthropic
> officially endorses `ln -s AGENTS.md CLAUDE.md`.
>
> **Windows checkouts:** run `git config core.symlinks=true` (or enable
> Developer Mode) or symlinks land as plain-text files containing the
> target path.
>
> **Gemini CLI** does not follow `GEMINI.md` symlinks
> ([gemini-cli#11547](https://github.com/google-gemini/gemini-cli/issues/11547),
> closed not-planned). If you use Gemini CLI on this repo, add
> `.gemini/settings.json` with
> `{"context":{"fileName":["AGENTS.md","CLAUDE.md","GEMINI.md"]}}`.

---

# Agent Operating Guide — ancplua-claude-plugins

> Sources: Boris Cherny (@bcherny) + Thariq (@trq212), 2026-04-16; Claude Opus 4.7 System Card (Anthropic, 2026-04-16,
> 232 pp.) — the most recent published card. Opus 4.8 builds directly on 4.7, so its safety and behavior findings still apply.  
> All System Card citations are to the April 16 2026 (4.7) edition; page numbers are stable. No 4.8 System Card has been published.

---

## Model & standard configuration

| Setting                                    | Value               | Why                                                                              |
|--------------------------------------------|---------------------|----------------------------------------------------------------------------------|
| Model                                      | Claude Opus 4.8     | —                                                                                |
| `effortLevel` / `CLAUDE_CODE_EFFORT_LEVEL` | `max`               | System Card p.192: "standard configuration: **adaptive thinking at max effort**" |
| `defaultMode`                              | `bypassPermissions` | Standing-authority repos; see Permission model below                             |
| `autoCompactEnabled`                       | `false`             | Every `/compact` is intentional — never silent                                   |
| `alwaysThinkingEnabled`                    | `true`              | Security property, not just quality (see Prompt injection below)                 |
| `agentPushNotifEnabled` + `voiceEnabled`   | `true`              | Leave long tasks running; recaps tell you what shipped and what's next           |

**Adaptive thinking** means the model determines per-query reasoning depth dynamically.  
`max` sets the ceiling; the model may use much less on a trivial query.
> System Card p.53: "the level of effort is **dynamically determined for each query by the model**"

**When to drop effort**: only for latency-constrained evals with wall-clock timeouts (the card ran Terminal-Bench 2.0
with thinking disabled for this reason, p.193). Interactive Claude Code sessions are not latency-constrained. Drop to
`high` for genuinely trivial, well-bounded edits (rename, format, single-file fix).

---

## Multi-agent architecture

```
Lead (this session)
  ├── continue            — same task; every token in window still load-bearing
  ├── rewind (esc-esc)    — wrong path; keep file reads, drop failed attempt
  ├── /compact <hint>     — mid-task bloat; steer the summary toward next direction
  ├── /clear + brief      — new task; hand-written context only, zero rot
  └── spawn subagent ────→ Task tool: own fresh window, returns conclusion only
```

**Spawn a subagent when** the next chunk will produce intermediate noise (file reads, greps,
dead ends) that the Lead will never need again. Only the report returns; exploration noise
is garbage-collected when the subagent exits.

**Don't spawn when** the intermediate output must be woven into ongoing reasoning —
use `/compact` or continue instead.

Mental test: *Will I need this tool output again, or just the conclusion?*

### Subagent patterns for this repo

| Task                                                 | Agent type            |
|------------------------------------------------------|-----------------------|
| Exploring a plugin's codebase for structure/patterns | Explore               |
| Verifying output against a spec or test suite        | general-purpose       |
| Writing docs from a git diff                         | general-purpose       |
| Reviewing upstream dependency changes                | general-purpose       |
| Security review of pending changes                   | security-review skill |

Context rot threshold for the 1M window: **~300–400k tokens** — task-dependent, not a
hard rule. File reads are the heavy hitter. Compact proactively, before the cliff edge.

---

## Known failure modes (System Card §6.2.1, p.95)

These are documented pilot-use findings for Opus 4.7 (which 4.8 builds on) in Claude Code and similar scaffolds.
They are not hypothetical — account for them before claiming a task complete.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ANcpLua/ancplua-claude-plugins](https://github.com/ANcpLua/ancplua-claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
