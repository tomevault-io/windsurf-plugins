---
trigger: always_on
description: Single source of truth for every coding agent in this repo (Codex CLI, Claude Code, Cursor, humans).
---

# tokenbroke — AGENTS.md

Single source of truth for every coding agent in this repo (Codex CLI, Claude Code, Cursor, humans).
`CLAUDE.md` is just `@AGENTS.md`; edit **this** file only. Read it fully before doing anything.

## 1. What this is

**One-liner:** A community leaderboard of the most rate-limited AI coding tool users alive, powered by a
zero-friction CLI that reads real local usage data for Codex CLI and Claude Code. A joke on the surface;
a public observability layer for AI coding tool rate limits underneath.

**Why it exists:** OpenAI's Codex (led publicly by Tibo, @thsottiaux) runs a recurring public loop: rate
limits degrade → community complains on X → team ships fixes and grants full usage resets. Anthropic's
Claude Code has similar rate-limit pain but no reset culture. tokenbroke instruments that loop: it
aggregates real usage telemetry from the community and turns it into legible, funny, screenshot-dense
public pressure, plus a genuinely useful "what is going on with Codex / Claude Code" dashboard.

**Endgame:** the labs themselves watch the dashboard.

## 2. Non-negotiables (read twice)

1. **Everything on the board is real.** The only way data enters the system is the CLI reading local
   files on the user's machine. No screenshot uploads, no manual entry, no "paste your numbers" form.
   Do not build one, even as a debug shortcut.
2. **Never gate submission behind auth.** First run is anonymous and instant. Claiming (GitHub OAuth,
   device-code style via a printed code) is optional vanity that attaches username + avatar.
3. **The name is a constant.** `tokenbroke`, `tokenbroke.lol`, `npx tokenbroke`, `~/.tokenbroke`, and
   `tokenbroke.lol/claim/<code>` come from `packages/shared/src/brand.ts`. Never hardcode them in copy,
   paths, or URLs.
4. **Individuals can be slightly fake; the aggregate must not be.** Anti-abuse protects aggregate
   stats, not individual rows. Cheating should be high-effort, low-payoff, statistically irrelevant.
5. **Privacy:** the CLI reads only usage/rate-limit state. It never reads or uploads prompts, code, or
   conversation content, and it submits only when the user runs the command. It never opens credential
   files (`~/.codex/auth.json`, `~/.claude/.credentials.json`), prompt history, memories, backups, or
   SQLite stores, and never calls a lab API with the user's tokens. Where a state file must be opened
   whole (`~/.claude.json` contains PII), the reader extracts only allowlisted fields and discards the
   rest. All filesystem access goes through one allowlist-enforcing layer that tests can instrument.
6. **Do not invent features** beyond this file. Ask.
7. **Ask before anything expensive to reverse:** package name, DB schema shape, auth provider, public
   API shape, hosting. There is no DB schema yet; do not add one without approval. Never publish to npm
   from a session. Never commit secrets.
8. **Tone:** deadpan infrastructure parody. Casual profanity fine. Funny in copy, dead serious in data
   integrity. **Never mean-spirited toward the lab teams**; the bit is affectionate pressure.

## 3. Naming

| Thing          | Value                                 |
| -------------- | ------------------------------------- |
| Product name   | `tokenbroke`                          |
| Domain         | `tokenbroke.lol`                      |
| CLI command    | `npx tokenbroke`                      |
| Config dir     | `~/.tokenbroke`                       |
| Claim URL      | `tokenbroke.lol/claim/<code>`         |
| npm package    | `tokenbroke` (available, unpublished) |

All of the above live in `packages/shared/src/brand.ts` (`BRAND`, `claimUrl()`) and are imported
everywhere: CLI output, site copy, OG cards. Cheap insurance against a rename.

## 4. The three layers (each feeds the next)

### Layer 1 — The Leaderboard (acquisition)
- Public ranked list of the most rate-limited developers.
- **Misery score** = f(remaining usage %, time until next reset). 0% remaining with 4 days to wait is #1.
- **Single trust lane:** every entry comes from the CLI reading real local data. "Everything on this
  board is real" is the brand.

### Layer 2 — The Sensor Network (the CLI; verification; data)
- `npx tokenbroke`: one command, ~5 seconds, zero signup.
- The votive-offering ritual: an individual submission is tiny but legible and public, and it visibly
  accumulates into collective pressure.

### Layer 3 — The Watch (retention)
- Reset radar with live countdowns in the viewer's local timezone.
- Days-since-last-reset counters per lab.
- Curated feed of rate-limit-relevant posts from Tibo and other team accounts.
- Aggregate telemetry dashboards: drain velocity, median remaining %, regression detection.
- North star: the single pane of glass for "what's happening with Codex and Claude Code."

## 5. CLI behavior (60% of the engineering, 100% of the product)

### First run, zero auth
1. **Detect + read local ground truth for BOTH tools when present.**
   - Claude Code: parse JSONL session logs under `~/.claude` (the ccusage approach) to reconstruct
     usage, remaining %, and drain history.
   - Codex CLI: read locally cached rate-limit/usage state from Codex's local files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SnehanChakravarthi/tokenbroke](https://github.com/SnehanChakravarthi/tokenbroke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
