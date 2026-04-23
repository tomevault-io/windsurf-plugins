---
trigger: always_on
description: > Keep this file concise. Only non-obvious decisions and rules belong here.
---

# CLAUDE.md

> Keep this file concise. Only non-obvious decisions and rules belong here.
> Don't add conventions, naming patterns, or project structure — those are discoverable from the code in seconds.

## Project Overview

Winzy.ai is a habit tracker with an optional social layer. Users track daily habits and optionally share progress with friends through **The Flame** — a visual consistency indicator. Friends can see your Flame and set experience-based challenges (grab coffee, play tennis).

**Core philosophy:**
- **Habit tracker first** — works beautifully standalone. No friends required.
- **Passive accountability** — knowing someone *can* see your Flame changes behavior. No messages, no nudges.
- **Consistency over streaks** — 60-day rolling window. Missing a day doesn't reset to zero.
- **Encourage, never punish** — "Ready to log today?" not "You haven't logged today!"
- **Zero-friction sharing** — public flame page at `winzy.ai/@username`, no account needed to view.
- **Shared experiences, not transactions** — challenge rewards are things you do TOGETHER.

**Product strategy:** Web-first PWA via Expo. Native apps later. Same codebase.

## Key Design Decisions

These are the non-obvious choices that agents get wrong without being told:

- **JetStream, not core NATS.** `user.deleted` cascades to 5 services that must all process it (data cleanup, GDPR). Core NATS is fire-and-forget — a restarting service silently misses the event.
- **Gateway validates JWT, services trust X-User-Id.** Services never re-validate tokens. The gateway strips any client-supplied `X-User-Id` and sets it from JWT claims. Never expose service ports directly.
- **Gateway must NOT reference Winzy.Common.** The gateway has no NATS dependency. If it needs shared types, reference `Winzy.Contracts` directly.
- **NATS subjects have no prefix.** `user.registered`, not `events.user.registered`. Matches `Subjects.cs` constants.
- **Result pattern for business logic errors.** Return result types, don't throw exceptions for expected failures (validation, not-found, conflict).
- **Gateway is exposed on port 5050, not 5000.** Port 5000 conflicts with macOS AirPlay Receiver. The gateway listens on 5000 internally, docker-compose maps `5050:5000`.
- **No direct DB access across services.** Each service owns its data. Cross-service data via REST or NATS only.
- **Internal endpoints stay internal.** Endpoints like `GET /habits/user/{userId}` are service-to-service only — never exposed through the Gateway.
- **Public endpoints are explicit.** Only these work without auth: `/auth/register`, `/auth/login`, `/auth/refresh`, `/habits/public/{username}`, `/notifications/vapid-public-key`.
- **Every service implements `GET /health`.** Returns service status, DB connectivity, and NATS connection status. Gateway aggregates all health checks.
- **Services return results, UI shows feedback.** Backend services never format user-facing messages. They return structured data; the frontend decides how to present it.

## Working Rules

- **Always read the relevant bead before reviewing or verifying work.** The beads contain acceptance criteria that define "done" — code review without checking the bead misses spec gaps.
- **No script-based mass changes.** Never run regex-based scripts to transform code files. Make changes manually or use parallel subagents for many simple changes.
- **No file proliferation.** Never create `V2`, `_improved`, `_new`, `_enhanced` variants of existing files. Edit in place. New files are only for genuinely new functionality that doesn't belong in any existing file.
- **Verify library APIs before implementing.** Use Context7 or search online to check current docs. Don't guess at API signatures or assume remembered usage is correct.

## Testing

Every test module must cover three areas:

1. **Happy path** — the expected workflow works correctly
2. **Edge cases** — empty input, boundary values, max limits, zero/null, concurrent access
3. **Error conditions** — invalid input, missing resources, network failures, unauthorized access

## Git Workflow

**All changes go through PRs. No direct pushes to `main`.**

- New task = new branch from main
- Branch naming: `feature/description` or `fix/description`
- Conventional commits. Include the beads issue ID when the commit maps to a specific issue.

### PR Workflow

**NEVER merge without explicit user approval.** After checks/review pass, ASK the user before merging.

1. Push branch, create PR: `gh pr create`
2. Wait for checks: `gh pr checks <number> --watch`
3. Read review: `gh pr view <number> --comments` — **NEVER skip this, even for non-code PRs**
4. Evaluate feedback critically — fix legit issues (bugs, security, logic), ignore noise (style nitpicks, "optional" suggestions)
5. Don't be afraid to amend when fixing issues that reviewers find in the previous commits
6. Push fixes, wait for re-review if needed. Repeat 2-4 until approved.
7. Before merge, review the commit stack: rebase to clean up fixups, drop noise (beads syncs, WIP), and make sure the remaining commits tell a clean, sensible story
8. **Ask the user for merge approval** — they may want to test or review the diff first
9. Merge and cleanup:
   ```bash
   gh pr merge <number> --rebase --delete-branch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gabko14) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
