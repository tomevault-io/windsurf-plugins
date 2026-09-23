---
trigger: always_on
description: Immortal Barons is a from-scratch Go clone of the 1990s BBS door game
---

# Immortal Barons — project guide

Immortal Barons is a from-scratch Go clone of the 1990s BBS door game
*Barren Realms Elite* (BRE). It is an independent reimplementation of the
game's rules and mechanics — no original code or art is used, and its prose is
its own. Never put John Dailey's (or any third party's) private contact info
into any artifact here.

**Where the line on the original's TEXT falls.** Its *prose* is not reused:
news lines, result reports, help and instruction text, flavor and narration are
all written here. What may match the original word for word is the **functional
furniture of a screen** — a question prompt, a field label, a menu item, a short
refusal — because those are dictated by what is being asked rather than by how
anyone chose to say it, and because IB's own versions were already all but
identical. The covert and terror operation menus were aligned that way on
2026-09-11 (`Send how many?`, `This will cost you N gold.  Accept?`,
`N agents sent out.`). Their two separate can't-afford refusals were aligned
with them and then dropped again on 2026-09-12: every site short of gold now
says the same thing in IB's own words, because three near-identical refusals in
two registers read as three different systems talking.

Two things this does NOT license, and both have bitten before: **code or art**
(source, decompiled routines, ANSI screens, logos) stays out entirely, and the
original's **product name** never goes on an IB screen — see below. When in
doubt about a string, ask: is this the game asking a question, or the game
telling a story? The first may match; the second is ours.

**Never put the original's NAME on IB's own screens.** Cloning a captured screen
copies its layout and colors; the product name in its header is branding, not
design, and must be replaced with IB's. The InterBBS Scores view shipped as
"Barren Realms Elite: Top Planets by Score" for months because the title came
along with the layout (fixed 2026-08-23; `TestScreensDoNotWearTheOriginalsName`
guards it). Naming the original in prose is fine and sometimes required — the
About screen's attribution and disclaimer, the README's Heritage, a doc
explaining a divergence. The line is identity, not mention.

A distinctive item name inside the game is a separate question, and Andy's to
answer one at a time. The nine pirate factions carry IB-original names; the
Gooie Kablooie, the S3-Sabre and SpyGuy keep the original's (#218). Ask before
renaming one, and do not revert one either way.

Be careful with other people's names and handles in repo artifacts (code,
comments, docs, commit messages, ChangeLog). The line is public vs. private:

- **Fine to name** — an actual project contributor; or a person credited for a
  **public document, project, or repository they have published** that we cite
  or build on (proper attribution of public work). The Heritage section names
  Mehul Patel, who created BRE, and John Dailey Software, which has owned it
  since 1998 — creating and owning are different roles, so do not credit either
  with the other's. Say **owned**, not maintained: no new BRE release is known
  of. Do NOT write the original off as abandoned either — the copyright is
  asserted for the current year, the games are still sold as registrations, and
  the site was posting news within days of this being written (checked
  2026-08-15).
- **Do NOT name** — usernames/handles from **private or community channels**
  (forum/Discord/Slack handles, private tips, relayed disassembly notes). Credit
  the *source* generically instead: "a disassembly of the original binary",
  "reconstructed from play data", "community strategy guides".

When unsure whether a source is public enough to credit by name, keep it
generic.

## This file, and the guides beside it

These notes are for any agent or tool working in this repository, kept
tool-neutral so they are not specific to one assistant. The root `CLAUDE.md` is
a one-line pointer at this file, because Claude Code does not discover
`AGENTS.md` on its own.

Task-specific guides live in **`.claude/skills/`**, one directory per topic,
each a `SKILL.md` whose `description` says when it applies. Read that
description to decide whether a guide is worth opening. The directory is named
for the tool that discovers them automatically, but the guides are plain
Markdown and are meant for any agent working here.

| Guide | Read it when |
| --- | --- |
| `bre-gather` | Reconstructing or checking any detail of the original — a menu, a constant, a screen layout. Read it BEFORE working from memory |
| `ib-testing` | Running the game to answer a question: verifying a number, reproducing something seen in play, exercising inter-BBS features |
| `ftn` | Anything touching FidoNet transport — packets, file attaches, BSO outbound, binkp, a mailer or tosser that is not delivering |
| `ansi-artwork` | Drawing or fixing text-mode art: a splash screen, a banner, a box border, colors that render wrong over telnet |

A guide describing a flag, path or behavior that no longer exists is worse
than no guide, because it will be believed. Fix one in place when you find it
stale.

## Build, test, run

```
go build ./...
go test ./...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andy5995/immortal-barons](https://github.com/andy5995/immortal-barons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
