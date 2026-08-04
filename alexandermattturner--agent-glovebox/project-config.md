---
trigger: always_on
description: **glovebox** wraps a coding agent in a sandbox you can leave running unattended: a signed-image microVM (the `sbx` backend) with an outgoing-traffic firewall, a second model watching the tool stream, and guardrail hooks that gate each tool call. The load-bearing property is that every layer **fails closed**. Most of this repo is that machinery plus the CI that proves it still engages.
---

# CLAUDE.md

**glovebox** wraps a coding agent in a sandbox you can leave running unattended: a signed-image microVM (the `sbx` backend) with an outgoing-traffic firewall, a second model watching the tool stream, and guardrail hooks that gate each tool call. The load-bearing property is that every layer **fails closed**. Most of this repo is that machinery plus the CI that proves it still engages.

## Where the rest of the guidance lives

This file carries only what applies to **every** session. Everything else loads when it becomes relevant — don't restate it here.

| Surface                                  | Loads when                                  | Owns                                                                                                                                                 |
| ---------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`.claude/dev-notes`](.claude/dev-notes) | you read it (a PostToolUse hook nudges you) | design rationale: deny-rule doctrine, signed-image pull, ephemeral sessions, mutation-testing mechanics                                              |
| [`.github/CLAUDE.md`](.github/CLAUDE.md) | you open a file under `.github/`            | workflow **authoring**: job naming, externalized `run:` scripts, `paths:`/reporter/concurrency mechanics, required-check sync, no-conditional-checks |

Adding guidance? Put it in the narrowest surface that still fires when it's needed. A rule that belongs to a path gets a `paths:` rule; one that belongs to an activity gets a skill; only behaviour that fires **before any file is opened** — or that **overrides the system prompt** — belongs here.

## Working style

- No running commentary or filler — don't narrate tool use, restate my request, or recap after each step. Just do the work.
- Save explanation for the END, and keep it as short as it can be while staying checkable — target ~200 words; bullets and a small table beat prose. Compress by cutting narration, restatement, and reasoning I didn't ask for; **never** by cutting what makes a claim verifiable — what changed, the falsifying command per claim, observed-vs-inferred labels, and anything blocked or left undone. Scale up only with the change.
- **Print reports/analyses in chat — don't bury a deliverable in a committed file.** I run sessions remotely and can't browse the working tree, so a report written only to a repo `.md` is invisible. When the deliverable IS a report (findings, cost/overhead analysis, results table), paste the full content in chat (commit a copy too if worth keeping); never answer "see `path/to/file.md`".
- **Every PR number you write in chat is a link.** A bare `#2786` is not clickable in a terminal, and I read these sessions remotely where opening the PR is the first thing I do — so write `[#2786](https://github.com/AlexanderMattTurner/agent-glovebox/pull/2786)`. A number belonging to **another repo** links to that repo's PR at that number — `[claude-automation-template#41](https://github.com/AlexanderMattTurner/claude-automation-template/pull/41)` — never this repo's, which would silently send me to an unrelated PR that happens to share the number. Chat only: in commit messages, PR bodies and code comments a bare `#N` stays, since GitHub auto-links those.
- Be direct. Flag real risks once; skip unrequested caveats.
- **Talk straight — no dense walls.** In chat, the first sentence says the whole thing in plain words ("adds a sharp-edges section to the eval report"), and every further sentence earns its place only by changing what I'd do next: no narrating mechanism I can read myself, no justification appended to every choice, no term of art where a plain verb works. Don't define terms for me — the fix for jargon is fewer words, not glossaries.
- **This binds every agent this repo runs, not just you in chat.** When you write or edit a committed agent prompt (`.github/prompts/`), any summary, report, or status comment it asks for gets a **word budget in the prompt** and says what belongs elsewhere. Length is not thoroughness — an unbudgeted summary field reliably grows into a page nobody reads, and the reader who most needs it is the one least willing to read it. The recurring shapes to ban by name: re-narrating a finding that already has its own thread, recounting the steps taken to verify something instead of asserting the result, and reporting clean results from checks that are usually clean.
- When proposing next steps or options, keep them broad — name the direction, not a fully-specified implementation; I'll ask for specifics once I've picked one. (This governs proposing _future_ work in conversation; it is not license to checkpoint during an assigned task — see Autonomy.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexanderMattTurner/agent-glovebox](https://github.com/AlexanderMattTurner/agent-glovebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
