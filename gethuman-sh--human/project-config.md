---
trigger: always_on
description: 'human' enables AI to act as human developers.
---

# Project

'human' enables AI to act as human developers.

Phase 1: Interact with issue trackers with product management issues and create implementation tickets with an implementation plan.

# Done Done

The status of 'done done' means not only things are done,
but nothing more needs to be done, e.g. change documentation,
website, configuration and the project is in the state for 
a new release.

Whatever you do, the new state needs to be 'done done'.

# Tickets

A ticket is **one artifact that evolves in place** through maturity stages (kinds):

1. **idea** — a raw thought, captured as a real ticket carrying the `human/idea` label (bare `idea` also classifies). Title-only is fine.
2. **pm** — promotion: the ideation agent rewrites title/description into product language and removes the idea label. Same key forever; PM ticket descriptions stay product language, no implementation detail.
3. **planned** — the engineering plan attaches to the ticket as a `[human:plan]` marker comment (full markdown; attach with `human marker post <KEY> plan --body-file -`, read it back with `human plan show <KEY>`). Re-planning posts a new plan comment; the latest wins.

**Topology rule:** whether planning ALSO creates a separate engineering ticket depends on the tracker config. Single-tracker is the default: unless a tracker carries an **explicit** `role: engineering` in `.humanconfig`, there is no second ticket — the plan comment on the ticket is the plan, and commits reference the one key. Split topology is opt-in: give a tracker an explicit `role: engineering` and planning then creates an engineering ticket on it whose description is the plan, with traceability running PM ticket → engineering ticket → git commits (reference the PM ticket in the engineering ticket, and both in commit messages). Role is never inferred from the tracker kind for the engineering side — a bare `linears:` entry with no `role:` stays single-tracker.

# Review handoff

When an engineer (human or AI agent) finishes coding an engineering ticket and `human-done` passes, the handoff to a reviewer goes via a structured comment on the **PM ticket**. This is tracker-agnostic (works on every backend `human` supports) and requires no custom tracker status.

Post it with `human handoff post <PM_KEY> [--engineering <KEYS>]` — the command derives the branch (current git branch), the commits (short SHAs referencing the work keys), and the daemon id, then verifies every commit is reachable on the branch before posting. Read it back with `human handoff show <PM_KEY>`. The posted body:

```
[human:ready-for-review]
engineering: HUM-89, HUM-90
branch: main
commits: 2037e40, 64bb370
```

- `engineering:` is comma-separated — one PM ticket can spawn multiple engineering tickets. **Single-tracker topology omits this line entirely**: the review target is the PM ticket the comment sits on.
- `branch:` is the branch the commits live on.
- `commits:` is the short SHAs attributed to the referenced keys (what `human commits for <KEY>` returns).

The `human-executor` agent posts this comment automatically as its final step. A reviewer (today: another user runs `/human-pickup-review <PM_KEY>`; future: daemon polling) reads the binding via `human handoff show`, runs `human-reviewer` against each engineering key (or against the PM key when the `engineering:` line is absent), and posts a `[human:review-complete]` follow-up marker (`human marker post`) on the same PM ticket with the verdict.

The TUI marks engineering tickets whose PM ticket carries an unresolved `[human:ready-for-review]` comment with an `(R)` annotation in the tracker view.

Do NOT use a custom tracker status for review signalling — that would require every team to reconfigure their tracker. Comments are the universal primitive.

# Daemon

When the human daemon is running, all CLI commands (except `daemon`, `install`, `init`, `tui`) are automatically forwarded to it. The daemon holds all tracker credentials on the host — **do NOT set tokens manually when the daemon is running**. Just run `human` commands directly.

The daemon is auto-discovered via `~/.human/daemon.json`. Check with `human daemon status`.

# Tracker Tokens (Daemon Host Setup)

These tokens only need to be set **once on the host where the daemon runs**. They are NOT needed for individual CLI invocations when the daemon is running.

## Preferred: Native vault provider (1Password)

Add a `vault` section and use `1pw://` references directly in `.humanconfig.yaml`:

```yaml
vault:
  provider: 1password
  account: my-account    # 1Password account name (top-left in app sidebar)

githubs:
  - name: personal
    token: 1pw://Development/GitHub PAT/token

linears:
  - name: work
    token: 1pw://Development/Linear Token/token

jiras:
  - name: amazingcto
    url: https://amazingcto.atlassian.net
    user: alice@example.com
    key: 1pw://Development/Jira API Key/token
```

Secrets are resolved via the 1Password desktop app integration. The 1Password app must be installed and running — it will prompt for biometric or master password authentication. Enable "Integrate with other apps" in 1Password Settings > Developer.

GitHub tokens can instead come straight from the GitHub CLI's keyring with a `gh://` reference — no PAT to copy anywhere:

```yaml
githubs:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gethuman-sh/human](https://github.com/gethuman-sh/human) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
