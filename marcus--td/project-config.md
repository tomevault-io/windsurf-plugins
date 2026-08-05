---
trigger: always_on
description: Run td usage --new-session at conversation start (or after /clear). This tells you what to work on next.
---

# AGENTS.md

## MANDATORY: Use td for Task Management

Run td usage --new-session at conversation start (or after /clear). This tells you what to work on next.

Sessions are automatic (based on terminal/agent context). Optional:
- td session "name" to label the current session
- td session --new to force a new session in the same context

Use td usage -q after first read.

## MANDATORY: Use `td` for Task Management

Run `td usage --new-session` at conversation start (or after /clear). This tells you what to work on next.

Sessions are automatic (based on your terminal/agent context). Optional:
- `td session "name"` to label the current session
- `td session --new` to force a new session in the same context

Sessions track implementers so the audit trail records who did what. Use a real reviewer sub-agent or a separate agent context for independent review — don't spin up throwaway sessions just to game the review check.

Use `td usage -q` after first read.

## Review Model (Trusted Review)

The default mode is now **`trusted`**. Trusted keeps the delegated review-attestation model — **prefer delegating review to an independent sub-agent** — and adds a flag-gated, audited self-review escape hatch for when delegation is not practical.

The rules:

- **Prefer an independent review.** A session that did not implement the issue reviews it (`td approve <id>`) or records an approval (`td approve <id> --record-only --reason "..."`). This is the norm; reach for it first.
- **Any session may perform the final close after approval.** Once an independent review exists, any session may run `td approve <id> --reason "..."` to close. The close is audited via `closed_by_session`; pass `--reason` if the closer is not the reviewer-of-record.
- **Self-review is allowed in trusted mode, but you must acknowledge it.** When you are the orchestrator/implementer and have already reviewed the diff yourself, approve+close with `td approve <id> --self-review --reason "..."`. The `--self-review` flag requires `--reason` and stamps `self_review` on the review row for audit. Do **not** fabricate a throwaway session to dodge the self-review acknowledgement — just acknowledge it.

### Modes (`review_policy_mode`)

Set with `td feature set review_policy_mode <mode>` (or `TD_FEATURE_REVIEW_POLICY_MODE=<mode>`).

- `trusted` — **default.** Delegated review-attestation plus a flag-gated, audited self-review escape hatch (`td approve --self-review --reason "..."`). Prefer delegation; self-review when delegation is impractical.
- `delegated` — review attestations + delegated close, with **no** self-review escape: the implementer cannot self-approve. Pin this for projects that want the hard wall.
- `balanced` — strict, plus a creator-approval exception with `--reason`. Legacy default for projects that set `balanced_review_policy=true`.
- `strict` — no prior involvement allowed at all.

### Orchestrator / Sub-Agent Flow

Preferred flow — orchestrator submits the issue for review, delegates the review to a sub-agent, then closes once the approval is recorded:

```bash
# Orchestrator creates work
td add "Refactor auth" --type feature

# Implementer sub-agent (separate session) does the work
td start td-a1b2
td log "implemented auth refactor"
td handoff td-a1b2 --done "refactor" --remaining "none"

# Orchestrator submits for review (this sets review_requested_by_session)
td review td-a1b2

# Reviewer sub-agent (separate session) records approval without closing
td approve td-a1b2 --record-only --reason "Reviewed diff, tests pass"

# Orchestrator, implementer, or another session closes using the recorded approval
td approve td-a1b2 --reason "Closing after recorded independent approval"
```

Trusted-mode shortcut — when you (orchestrator/implementer) have reviewed the diff yourself and delegation is not practical, acknowledge the self-review instead of spawning a reviewer session:

```bash
td review td-a1b2
td approve td-a1b2 --self-review --reason "Reviewed diff myself, tests pass"
```

The reviewer (when delegated) must be independent; the closer is recorded separately for audit. A self-review is recorded as such.

## Development Approach

- Be pragmatic: `td` is a focused local tool, not an enterprise platform.
- Use worktrees for major features, risky migrations, or parallel work. Make
  quick localized fixes on the current branch when scope and verification are
  clear.
- One independent review and one rejection cycle is normally enough. Continue
  only for a genuine P0 data-loss/security finding; track other observations as
  follow-up work.
- Test likely failures and important boundaries, not exhaustive hypothetical
  states without a concrete use case.
- Surface friction after the first surprising blocker or material scope growth.
  Pause and explain it rather than silently turning a small fix into a subsystem.

## Build & Install

```bash
go build -o td .           # Build locally
go test ./...              # Test all
```

## Version & Release

```bash
# Commit changes with proper message
git add .
git commit -m "feat: description of changes

Details here

🤖 Generated with Codex

Co-Authored-By: Codex Haiku 4.5 <noreply@anthropic.com>"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcus/td](https://github.com/marcus/td) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
