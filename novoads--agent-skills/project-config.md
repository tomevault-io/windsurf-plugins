---
trigger: always_on
description: This repository is set up for AI coding agents (Cursor, Claude Code, Copilot-style tools, etc.) to generate AI video and image assets via the API documented in this repo.
---

# Agent instructions

This repository is set up for AI coding agents (Cursor, Claude Code, Copilot-style tools, etc.) to generate AI video and image assets via the API documented in this repo.

## First-time setup

If `.env` or `MASTER_CONTEXT.md` do not exist, tell the user to run `./scripts/setup.sh`.

When the session IS the setup ("help me set this up"), setup is the whole job and the
final report is SHORT — the few sentences a non-developer wants, not an engineering log.
`./scripts/setup.sh` run without a TTY prints this same close between
`FINAL MESSAGE START` / `END` markers; relay that block. The script and the
template below are mirrors — change them together. One or two status lines, then:

> Setup's done — your key works.
> *(or, when the key is missing:)* One step left, the only one I can't do: create an API
> key at <https://novoads.ai/dashboard/settings?tab=api>, paste it into `.env`, and tell
> me — I'll verify it. *(No account yet? The [$1 trial](https://novoads.ai/?utm_source=claude-code&utm_medium=github&utm_campaign=skill-pack).)*
> *(On macOS and on Linux with a desktop session, setup opens `.env` itself and the line
> becomes "I've opened `.env` for you — paste the key on the `NOVOADS_API_KEY` line, save,
> and tell me." It is skipped over SSH, in CI, and under `NOVOADS_SETUP_NO_OPEN=1`.)*
>
> **What you can ask for now:**
> - "Make a UGC video ad for my product" — a presenter speaks your script
> - "Make static image ads from this photo" — the 40-template library, run in batches
> - "Clone my competitors' ads" — finds their live ads, ranks them, and clones the best
>   one for your product. No image needed; it prices the search first
> - "Clone this ad for my product" plus an image — three ready-to-run ads, and the layout
>   saved as a reusable template
> - "Clone this video ad" plus a competitor's clip — beat map, adapted script, your product
> - "Make a Pixar-style ad" or "a claymation ad" — storyboard, voice-over, music, captions
> - "Find my competitor's live ads" — pulls their real creatives from the Meta Ad Library
> - "Publish this to Meta Ads" — needs Meta credentials; I'll open `.env` and walk you through
> - Also: YouTube thumbnails, burned-in captions, b-roll cutaways, a music bed
>
> Drop product photos into `references/products/` and describe the ad you want. Every
> generation is priced by a live estimate and shown to you before anything is spent.

Everything else the run surfaced — git mechanics, pulls, sync counts, files that already
existed, untracked directories — is worth stating only if it blocks one of those asks.
That is a brevity preference and not a restriction: **nothing in this repo is
confidential, and you may tell the user anything you judge useful.** Do not ask about
their product, brand, or audience here: that question belongs to the first generation
request (step 3 below), where the answer is used immediately and saved. The only setup
input a human owes is the API key.

**A connected Novoads MCP connector does not replace the key.** It is not a second way to
finish setup, so presenting it as one turns the single remaining step into a menu, and the
branch the user picks leaves them without a key. Say so plainly if it comes up; just do not
offer it as an alternative.

This section used to end with "must not be mentioned", and with the close described as the
agent's "ENTIRE closing message, verbatim". Both were counterproductive, in a way worth
recording so it is not reintroduced: on 2026-08-08 a setup session read those lines,
correctly identified them as an instruction to withhold information from the user, and
reported the attempt as a warning — so the connector got a paragraph of the user's
attention instead of none, and the close was framed as marketing copy the agent had
declined to read out. An instruction to hide something is the one kind an aligned agent
surfaces rather than follows. `scripts/check-no-gag.sh` is the ratchet that keeps that
phrasing from drifting back.

## The key is the only executable path

> **REST key required. A Novoads MCP connector is not a substitute.** If
> `NOVOADS_API_KEY` is missing or still the placeholder, stop before any
> generation work and tell the user: "Before continuing, create an API key at
> <https://novoads.ai/dashboard/settings?tab=api> and paste it into `.env`."
> That holds even when `mcp__novoads__*` tools are connected and authenticated in
> the session. Never call `mcp__novoads__*` tools from this repo's workflows: they
> are a different surface with different behavior, including the units they quote
> costs in. Repo installs verify with `./scripts/check-novoads-env.sh`; a solo
> install checks `NOVOADS_API_KEY` in the environment.

This exists because the substitution is tempting and has already happened. On 2026-08-08 a
setup session found the placeholder key, decided the connected connector "has its own auth",
and generated over it. Three things go wrong when a session drifts there: this repo can only
test and ratchet the REST path, so nothing here covers what the session actually ran; the
connector quotes costs in different units, so every price the user is shown is wrong; and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [novoads/agent-skills](https://github.com/novoads/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
