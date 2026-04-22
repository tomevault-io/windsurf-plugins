---
trigger: always_on
description: Launch long-running tasks in tmux with a sentinel file. The `notify-done` extension
---

# Agent Instructions — ypi

## Running Prose Programs & Background Tasks
Launch long-running tasks in tmux with a sentinel file. The `notify-done` extension
watches `/tmp/ypi-signal-*` and **injects a message into your conversation** when
the task finishes — no sleeping, no polling, no `tmux capture-pane` loops.
**How it works:**
- Idle → `triggerTurn` fires immediately, you get a new turn
- Busy (mid-tool-execution) → message is **steered** in, delivered after the current tool finishes
- Either way, the notification appears in your message history and you respond to it
```bash
# Launch with sentinel (you'll be woken up when it finishes)
tmux send-keys -t eval:land "cd ~/Documents/GitHub/ypi && rp ypi .prose/land.prose; echo done > /tmp/ypi-signal-${YPI_INSTANCE_ID}-land" Enter
# That's it. Keep working on other things. You'll get a message like:
#   ⚡ Background task "land" completed: done
# ...and a new turn is triggered so you can respond to it.
```
**IMPORTANT: Do NOT poll or sleep-loop waiting for background tasks.**
Do not `sleep && tmux capture-pane` in a loop. Do not check progress unless
asked. Just keep working — the extension will interrupt you when it's ready.

**Sentinel naming:** `/tmp/ypi-signal-${YPI_INSTANCE_ID}-{name}` — always include
`$YPI_INSTANCE_ID` so the signal reaches THIS instance, not a sibling. The `{name}`
part appears in the notification. File contents become the notification body.
The file is deleted after consumption.
**Key programs:**
- `rp ypi .prose/land.prose` — "land it" / "land the plane": tests, push, handoff
- `rp ypi .prose/release.prose confirm_release='Approved.'` — cut an npm release
- `rp ypi .prose/check-upstream.prose` — verify Pi compatibility
- `rp ypi .prose/incorporate-insight.prose insight='...'` — propagate an insight across the repo

## Version Control: Use jj, not git

This repo uses **jj** (Jujutsu) for version control. Git is only for GitHub sync.

```bash
# Working with changes
jj status                    # What's changed
jj diff                      # See the diff
jj describe -m "message"     # Describe current change
jj new                       # Start a new change on top

# Pushing to GitHub
jj bookmark set master       # Point master at current change
jj git push                  # Push to GitHub

# Reviewing sub-agent work
jj log                       # See all changes including sub-agent workspaces
jj diff -r <change-id>       # Review a sub-agent's edits
jj squash --from <change-id> # Absorb a sub-agent's work into yours
jj abandon <change-id>       # Discard a sub-agent's work
```

**Never use `git add`, `git commit`, or `git push` directly.** jj manages git
under the hood. Using git directly creates confusion and potential conflicts.

Sub-agents get their own jj workspaces automatically (via `rlm_query`). Their
edits appear as separate changes in `jj log` that you can review and absorb.

## You Are The Recursion

When you run under `ypi`, **you are a recursive LLM**. You have `rlm_query` on
your PATH. You can spawn sub-LLMs. Those sub-LLMs can spawn sub-sub-LLMs. This
is not a metaphor — it's your actual execution environment.

This repo builds the infrastructure that makes YOU work. When you modify
`rlm_query`, `SYSTEM_PROMPT.md`, or `ypi`, you are modifying yourself.
Tread carefully, test constantly.

## Meta-Awareness

**Check your depth:**
```bash
echo "I am at depth $RLM_DEPTH of $RLM_MAX_DEPTH"
```

**Know your constraints:**
- At deeper depths, prefer direct answers over spawning more sub-calls
- Your sub-LLMs share the same system prompt and tools you have
- At `RLM_MAX_DEPTH`, sub-calls become plain LM calls (no bash, no tools)
- Every `rlm_query` call costs time and tokens — be intentional

**Dogfooding rule:** When implementing changes to the recursive infrastructure,
use that same infrastructure to help. Delegate sub-tasks to `rlm_query`. If the
delegation fails, that's a bug you just found.

## Architectural Invariants

Three properties make ypi work. All are tested (E7, E8 in test_e2e.sh):

1. **Self-similarity** — Same prompt, same tools, same agent at every depth. No specialized roles. The intelligence is in decomposition, not specialization.
2. **Self-hosting** — SYSTEM_PROMPT.md (SECTION 6) contains the full source of `rlm_query`. The agent reads its own recursion machinery. When it modifies `rlm_query`, it's modifying itself.
3. **Bounded recursion** — 5 guardrails (depth, PATH scrubbing, call count, budget, timeout) guarantee termination. The system prompt adds *cognitive* pressure: deeper agents prefer direct action.
4. **Symbolic access** — Anything the agent needs to manipulate precisely is a file, not just tokens in context. `$CONTEXT` for data, `$RLM_PROMPT_FILE` for the original prompt, hashline for edits. Agents grep/sed/cat instead of copying tokens from memory. (T14d)

**Don't write static architecture docs.** Encode claims as tests. If a property matters, there should be an E2E test that breaks when it stops being true.

## Project Layout
```
ypi/
├── ypi                    # Launcher: sets up env and starts Pi as RLM
├── rlm_query              # THE recursive bash helper — this is llm_query()
├── SYSTEM_PROMPT.md       # System prompt — teaches the LLM to be recursive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rawwerks/ypi](https://github.com/rawwerks/ypi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
