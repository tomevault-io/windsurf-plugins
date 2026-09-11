---
trigger: always_on
description: Incident management platform built with Rails 8.1. Currently integrates with Slack, designed for multi-platform support (Teams, etc.).
---

# Firefight

Incident management platform built with Rails 8.1. Currently integrates with Slack, designed for multi-platform support (Teams, etc.).

## CI

Run `bin/ci` to validate changes. It runs rubocop, archspec (architecture boundaries, rules in `Archspec.rb`), bundler-audit, brakeman, rails test (parallel), system tests, and seeds.

`archspec_todo.yml` is empty and stays empty. Never add an entry to get a build green: a new boundary violation means the code is in the wrong place. The `archspec` GitHub job fails on any violation and on a non-empty todo file, and it must be a required check once branch protection is available on the repo.

ArchSpec proves the named boundaries. The leaks that hide behind an allowed name (a platform's shape in a column, a permission rule inlined in a controller, logic in an entry point, a TypeScript mirror of a Ruby list) are caught by the **boundary review**: run `/boundary-review` on the diff before opening any PR that touches `app/`, `engines/`, `lib/`, or `app/frontend/`, fix what it finds, and end the PR body with its report. A PR without that report is not ready.

## Git & PRs

- **Never merge a PR without an explicit instruction to merge in the current message.** Opening PRs when asked to build something is fine. Merging is always the user's call, every time. Prior merge approvals and inferred intent (e.g. a bug report that a feature "isn't working") do not count.

## Product decisions are the user's (always applies)

The user decides how the app behaves. You suggest, they choose. This is not a
preference, it is the working agreement.

- **Never silently skip, defer, or narrow anything.** No shortcuts, no MVP, no
  proof of concept, no "good enough for now". This is production software with
  great UX or it is not shipped.
- **Any choice that changes what a user sees or experiences is theirs to make.**
  Whether a Slack message posts, whether a button appears, whether a field is
  required, what copy says, what a transition does downstream. If you find
  yourself reasoning "there's nothing to follow up here so I'll skip it", stop
  and ask instead.
- **Scoping something out is itself a decision.** "That's a separate piece of
  work" is a suggestion, never a conclusion. Say it and wait.
- **Silence reads as complete.** If you did not do part of something, say so in
  the same message, before they find it.
- Applies to omissions as much as additions. The bug they cannot see is the one
  you decided not to build.
- **Never change existing behaviour that was not asked for. Not as a side
  effect, not to resolve an inconsistency you noticed, not because the new
  rule "needed a value" for the neighbouring case.** If implementing the ask
  forces a choice about anything adjacent, keep the adjacent behaviour exactly
  as it is and ask, before writing it. Flagging the change afterwards in a PR
  body or a summary does not make it allowed. This has happened once already:
  a guard against adding actions to a resolved incident was written so that
  follow-ups became addable during a live incident on the dashboard, which
  nobody asked for.

## Documentation (always applies)

Product docs live in a separate repo, `../firefight-landing`, and are served at `firefight.app/docs`. They are part of the change, not a follow-up.

- **Any change a user can see requires a matching docs update.** New or changed `/ff` commands, dialogs, settings screens, API endpoints, MCP tools, webhook events, or renamed navigation. A feature that ships undocumented is half-shipped.
- **`../firefight-landing/CLAUDE.md` owns how docs are written**, covering audience, voice, punctuation, page shape and sidebar wiring. Read it before touching `src/content/docs/`, and follow it over any instinct carried across from this repo.
- **Docs ship as their own PR** in that repo, opened alongside the code PR here, with the code PR naming it.
- Repo-internal docs under `docs/` are engineering references and a separate obligation: update the relevant one in the same PR as the code.
- If a change turns out to need no docs update, say so explicitly rather than leaving it unmentioned.

## No shortcuts (always applies)

Production-grade or not at all. Every one of these has already been violated once; none is hypothetical.

- **Never ship a placeholder interaction.** No `window.prompt` / `confirm` / `alert`, no unstyled control, no "fine for now" input. If a flow needs input it gets the same dialog treatment as every other flow in the app. There is no such thing as an incidental piece of UI. The throwaway bit is usually the first thing a user touches.
- **Reuse the existing pattern before inventing one.** Find the nearest component already in `app/frontend/` and match it. Introducing a new interaction pattern is a decision to state out loud, never a side effect of moving fast.
- **If the model supports N, the UI must not assume 1.** Rendering `records.find(...)` where the schema permits many silently hides rows. Check the second case: the second connection, the second credential set, the already-connected state, the empty list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FireFightLabs/firefight](https://github.com/FireFightLabs/firefight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
