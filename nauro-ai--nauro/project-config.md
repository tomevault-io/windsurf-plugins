---
trigger: always_on
description: Writes durable shared context into Nauro's project store so other agents (a later session or a parallel one) can discover and pull it, finds and reads context another agent left, or captures a resumable brief so your own next session in this environment picks up cleanly. Three modes. Author writes a shared brief for any agent. Find locates and reads a brief another agent left. Resume captures a self-directed brief and hands back a short prompt to start the next session. Offer Resume mode when th
---


# Nauro context skill

Write durable shared context into Nauro's project store so other agents can discover and pull it, find context another agent left, or capture a resumable brief so your own next session in this environment picks up cleanly. The skill has three modes. Author writes a shared brief for any agent. Find locates and reads a brief another agent left. Resume captures a self-directed brief for your next session and hands you a short prompt to start it. Author and Resume use the agent's filesystem write and the `nauro status` shell command to resolve the store path, alongside the MCP tools `get_context`, `get_raw_file`, and `flag_question`. The skill never files a decision.

A brief is free-form working context that is not yet a decision: a migration's half-finished state, a research synthesis, an investigation's findings, a map of a subsystem, or the in-flight state your next session must reconstruct. Decisions remain the formal record; briefs are the connective tissue between them.

## Picking the mode

Read the invoking prompt to pick the mode. Resume mode fits a same-environment continuation request, in the user's own words: give me a prompt for a fresh session or instance, output or draft the prompt, hand off this work, write a resume doc. When a request reads that way, offer Resume mode and let the user accept before running it; never substitute it silently for what the user asked. Author mode fits a request to share context for other agents ("write this up for the other agents", "leave a brief on the auth migration"). Find mode fits a request to read what prior agents shared ("is there a brief on this?", "pull any shared context before you start").

Three shapes route elsewhere and the skill says so rather than forcing a fit: a request to forward a mission to a worker agent with the parent session still live belongs in Author mode with the durable payload under `context/`; a request to hand work to a store-blind surface such as a Codex consult keeps the manual paste-the-prompt ritual, which remains correct there; and a request that is genuinely ambiguous gets a one-line question before anything runs.

Every mode's write path goes through the local store: the agent writes the brief to the local store on disk, then `nauro sync` pushes it. A pure chat surface with no local store cannot write an arbitrary store file, so chat-only authoring and resume capture are out of scope; chat surfaces can still read briefs via `get_raw_file`. Pass `project_id` explicitly on every MCP call when more than one project exists, matching the adopt-skill convention.

## Step 1 — Author: write the brief file

The agent writes the brief body to `<store>/context/<slug>.md` using its own filesystem write. Resolve `<store>` by running `nauro status`, which prints the absolute store path; the store lives at `~/.nauro/projects/<id>/`, outside any repo, so it cannot be guessed from the working directory. The CLI push enumerates the whole store, so a file under `context/` syncs with no code change.

The slug is `<origin>-<topic>-<YYYYMMDD>-<short-uid>`, for example `codex-auth-migration-20260605-h7k2`. `<origin>` is your surface or agent tag, `<topic>` is a short kebab-case subject, `<YYYYMMDD>` is today's date, and `<short-uid>` is a few random or session-derived characters. The short-uid is load-bearing: two agents on separate machines reconcile only at the shared store, so entropy in the slug — not a lock — is what keeps their briefs from colliding. Briefs accumulate append-only under `context/` — never overwrite or delete an existing brief. If the chosen slug already exists, add a disambiguator rather than replacing it.

The brief opens with YAML frontmatter. Required: `author` (your surface or agent tag), `created` (today's date), and `summary` (one line). Optional: `for` (the intended audience), `surface` (where it was authored), and `status`. The `author` field is advisory and unverified — it is self-asserted provenance, never a trust signal, and `surface` is descriptive only, never a discovery or merge key. Keep the whole file under `MAX_BRIEF_BYTES` (50 KiB); real briefs run well under that.

## Step 2 — Author: flag the discovery pointer

The agent calls `flag_question(question="BRIEF: context/<slug>.md — <one-line summary>")`. This flagged question is how other agents discover the brief. It lives in `open-questions.md`, which is set-union-merged on sync, so pointers from concurrent authors all survive. A shared index file is deliberately not used: it would not be union-merged, so concurrent appends would be lost under last-writer-wins. The `BRIEF:` marker text is literal so the Find flow can locate it.

## Step 3 — Author: sync, branching on linkage

The agent runs `nauro status` to read the project's linkage, then branches:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nauro-AI/nauro](https://github.com/Nauro-AI/nauro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
