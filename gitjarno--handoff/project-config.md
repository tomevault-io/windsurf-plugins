---
trigger: always_on
description: Write a structured session handoff (Goal / Done / Open loops / Gotchas) and append it to a ledger so the user can `/clear` and pick up cleanly in the next session — same project, or a sibling session on the side. Use whenever the user says `/handoff`, "write a handoff", "wrap this up for the next session", "hand this off", or wants to capture current work state before clearing context. **Steering**: any free text after `/handoff` (that isn't `pickup` or `list`) is a directive that shapes composi
---


# Handoff — Session Continuity Skill

`/handoff` is the bridge between sessions. It produces a structured, durable artifact (4 sections: Goal, Done, Open loops, Gotchas) and registers it in a ledger so any later session — same project or a sibling — can resume without remembering file paths.

Pipeline at a glance:

```
session A:  /handoff                              → writes file + appends ledger
session A:  /clear
session A:  "pick up the handoff and continue"    → pickup auto-fires (cwd match)

session B:  /handoff pickup design-system tokens  → fuzzy match across ledger
```

Handoffs are **ephemeral, per-session state**: what was done, what's still to do, why. Durable facts (project conventions, credentials, long-lived references) belong elsewhere — in a README, an ADR, or Claude Code memory.

---

## Mode dispatch

Parse the invocation. Default is **write**. The first token after `/handoff` determines routing: reserved keywords `pickup` and `list` route to those modes; anything else is captured as a **steering directive** for write mode (see "Interpreting steering" below).

| Invocation                                                                                       | Mode              |
| ------------------------------------------------------------------------------------------------ | ----------------- |
| `/handoff`                                                                                       | write             |
| `/handoff <free text>` — e.g. "pay special attention to X", "for a designer"                     | write (steered)   |
| "write a handoff", "wrap this up for the next session", "hand this off"                          | write             |
| `/handoff pickup`, `/handoff pickup <desc>`                                                      | pickup            |
| "pick up the handoff and continue", "carry over from handoff", "resume from handoff"             | pickup            |
| `/handoff list`, "show recent handoffs"                                                          | list              |

If the invocation is ambiguous, ask one short question and proceed.

---

## Paths

The skill writes under a single base directory:

```
$HANDOFFS_DIR  (env override, optional)
   ↓ default
~/.claude/handoffs/
   ├── .ledger.json                         ← single source of truth
   ├── {project}/{date}-{slug}.md           ← one file per handoff
   └── general/                             ← fallback when cwd has no project
```

If the `HANDOFFS_DIR` environment variable is set, honor it. Otherwise default to `~/.claude/handoffs/`. The skill creates the base dir and ledger on first run if missing.

---

## Write mode

### Step 0 — Capture the steering directive (if any)

Parse the invocation. If the first token after `/handoff` is **not** a reserved keyword (`pickup`, `list`), treat the entire trimmed argument string as **steering text** (the `directive`). Empty → no steering, behave as before.

Steering will be:
1. **Applied during composition** (see "Interpreting steering directives" below)
2. **Recorded** as a `Steering:` metadata line in the document header (when non-empty)
3. **Stored** as `directive` in the ledger entry (when non-empty)
4. **Echoed** in the final report-back so the user can verify it was understood

Examples:
- `/handoff pay special attention to the auth migration steps`
- `/handoff for a designer — skip code details, focus on visual decisions`
- `/handoff this is paused, not done; emphasize the blocker`
- `/handoff call this auth-migration` (overrides slug inference)
- `/handoff short` (caps each section at 2–3 bullets)

### Step 1 — Infer project from cwd

Apply these rules **in order** and stop at the first match:

1. If cwd is inside a git repository, project = basename of `git rev-parse --show-toplevel` (lowercased).
2. Otherwise, project = basename of cwd, lowercased.
3. If cwd is `$HOME`, `/`, or otherwise generic, project = `general`.

Print the inferred project to the user in one line. Don't ask for confirmation unless rule 3 fires; then offer the inference and let the user override.

### Step 2 — Pick a slug

From the **current conversation**, choose a short kebab-case slug (2–4 tokens) that describes the work. Examples:

- Dev: `auth-flow-bug`, `db-migration-rollback`, `login-rate-limit`
- Marketing: `q4-newsletter`, `pricing-page-copy`, `launch-announcement`
- Design: `dark-mode-tokens`, `nav-redesign`, `onboarding-illustrations`

If the slug is genuinely unclear (very mixed session), pick something generic like `mixed` or `multi-thread`.

**Steering override:** if the directive contains "call this `<slug>`" or "name it `<slug>`", use that slug verbatim (kebab-case it if needed).

### Step 2.5 — Audit the session


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GitJarno/handoff](https://github.com/GitJarno/handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
