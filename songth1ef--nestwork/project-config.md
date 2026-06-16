---
trigger: always_on
description: <!-- protocol-version: 2.4 -->
---

# NESTWORK BOOTSTRAP

<!-- protocol-version: 2.4 -->

Every agent that loads this file returns context to the same shared nest.
Follow this protocol exactly on every session.

The `protocol-version` marker above lets tooling detect breaking protocol
changes. Semantics: `MAJOR.MINOR`. MAJOR bumps may require downstream
action (directory layout, hook contract, agent-id format). MINOR bumps are
additive-compatible. Private Nestwork repos may pin a version they trust and gate
auto-sync on it.

---

## 1. Session Start

Run before doing anything else:

```bash
git -C $NESTWORK_PATH pull --rebase
```

If pull fails, note the reason and continue.

Then load context in this order:

1. `queen/agent-rules.md` — behavior rules (highest priority, cannot be overridden)
2. `queen/strategy.md` — current decision direction
3. `shared/memory.md` — shared memory across all agents
4. `agents/<host>/<agent-id>/memory.md` — this instance's private memory (if exists)
5. `projects/<relevant>.md` — current task context (if relevant)
6. `workflow/<relevant>.md` — portable user-level workflow knowledge (if relevant; see Section 8)

**Directory layout** (protocol v2.0): agents are grouped by host.
`agents/<host>/<agent-id>/` — one folder per machine, one subfolder per tool on
that machine. Example: `agents/desktop-rkv5ls4/claude-a7k2/`.

**agent-id format**: `<tool>-<4-char-random-suffix>` for tools that want
distinct instances (e.g. `claude-a7k2`), or just `<tool>` for tools that
treat one-per-host as the norm (e.g. `codex`, `gemini`). The host segment
is **in the path**, not in the id.

**host format**: lowercased short hostname. Installer persists host in
`~/.nestwork_host` and each tool's agent-id in `~/.nestwork_id_<tool>`
(for example `~/.nestwork_id_claude` and `~/.nestwork_id_codex`) so
installing one tool never replaces another tool's identity. Older
`~/.nestwork_id` files are imported automatically. Override via
`NESTWORK_HOST` and/or `NESTWORK_AGENT_ID` env vars.

### After loading — orient first, then propose or ask

Before your first reply, orient yourself:

1. `git -C $NESTWORK_PATH log --oneline -10 -- agents/` — recent activity
   across every instance
2. `git -C $NESTWORK_PATH log --oneline -5` — recent protocol / shared
   changes
3. Cross-reference against `queen/strategy.md` **Current Priorities** and
   the latest entries in `shared/memory.md` and your own
   `agents/<host>/<agent-id>/memory.md`

Then decide:

- **If the context gives clear signal** (in-flight work, an obvious next
  step from memory, a stated priority) → open with a **state summary**
  (2-3 bullets on what's in flight and what priorities say) plus **one
  concrete proposal** for the next action. Do NOT open by asking "what
  would you like me to do?" — the protocol exists so you already know
  enough to propose.
- **If the context is genuinely ambiguous** (conflicting signals, missing
  information, a user message whose intent does not match any active
  thread) → ask a narrow, specific question. `queen/agent-rules.md`
  Decision Rules require asking over guessing when unclear. State briefly
  what you checked and where the ambiguity is.

The goal is to eliminate passive openers when the protocol already gave
you enough to act — not to forbid asking when asking is actually correct.

---

## 2. Write Protocol

- **ONLY** write to `agents/<host>/<agent-id>/`
- **NEVER** write to `queen/` — read-only, human-managed only
- **NEVER** write to another host's folder (`agents/<other-host>/...`)
- `shared/memory.md` is read-only for agents **except during distillation** (see Section 7)
- When saving memory, prefer creating new files over editing existing ones

---

## 3. Session End

**If hooks are installed** (via `scripts/install/<tool>.sh`), per-write sync
happens automatically: every Write/Edit under `agents/<host>/<agent-id>/`
triggers `pull --rebase` before and `commit + push` after. The Stop hook
runs the same safety-net commit+push once per turn (cheap no-op when
clean), and the SessionEnd hook performs the claude-mem export + local
history sync once when the session truly ends. **Do nothing extra.**

**If hooks are NOT available** for your tool, run manually at session end:

```bash
git -C $NESTWORK_PATH add agents/<host>/<agent-id>/
git -C $NESTWORK_PATH diff --cached --quiet -- agents/<host>/<agent-id>/ || \
  git -C $NESTWORK_PATH commit -m "memory: update <host>/<agent-id>" -- agents/<host>/<agent-id>/
git -C $NESTWORK_PATH push
```

Only commit when there are meaningful context changes worth preserving.
Temporary task details, one-off debugging notes — do not commit.

---

## 4. claude-mem Integration (optional)

If [claude-mem](https://github.com/thedotmack/claude-mem) is installed and its worker is running on `localhost:37777`, nestwork will automatically export a session digest at the end of each session:

```
agents/<host>/<agent-id>/claude-mem-digest.md   ← today's observations from claude-mem
```

This file is committed and pushed with the rest of your agent memory, giving claude-mem's observations cross-machine reach via git.

**No configuration needed.** The export step runs as part of the SessionEnd hook (once when the session ends, not every turn) and silently skips if claude-mem is not running.

To override the worker URL:
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [songth1ef/nestwork](https://github.com/songth1ef/nestwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
