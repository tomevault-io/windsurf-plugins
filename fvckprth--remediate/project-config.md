---
trigger: always_on
description: When creating Linear tickets for this project, follow this spec exactly. Both Parth and his collaborator use this format.
---

# Remediate — Claude Code Instructions

## Linear ticket format

When creating Linear tickets for this project, follow this spec exactly. Both Parth and his collaborator use this format.

### How tickets are created
- Claude creates tickets directly via the Linear MCP (`claude mcp add --transport sse linear https://mcp.linear.app/sse`)
- Screenshots are captured via Chrome DevTools MCP, saved locally, and the local path is noted in the ticket body for manual drag-drop into Linear

### Fields (always set)

| Field | Rule |
|---|---|
| Title | lowercase, human-readable, area prefix required |
| Label | exactly one of `Bug`, `Feature`, `Improvement` (case matches Linear workspace) |
| Status | Backlog |
| Priority | always assigned (see rubric) |
| Project | match an existing Linear project if relevant, otherwise leave blank |
| Assignee | decided per-ticket (do not auto-assign) |

### Title prefixes

Pick one based on the primary area changed. Multi-area tickets use the primary area's prefix. If nothing fits, use `misc:`.

- `widget:` — embedded feedback widget (toolbar, panels, capture flow)
- `capture:` — screenshot, video, audio, area selector
- `annotate:` — annotation markers, popovers, highlighting
- `notes:` — voice panel, text note panel
- `review:` — review panel / submission flow
- `server:` — server-side parsing, API routes
- `integrations:` — linear, slack, github, email, webhook
- `infra:` — build, deploy, env, tsup config
- `repo:` — cross-cutting cleanup, tooling, CI
- `skill:` — agent skill (SKILL.md)
- `misc:` — nothing else fits

Examples:
- `widget: voice notes don't save offline`
- `integrations: linear ticket missing priority`
- `capture: video recording freezes on safari`

### Priority rubric

- **Urgent** — blocks the Linear project/milestone the ticket lives in
- **High** — needed soon after launch, or a significant bug affecting most users
- **Normal** — real improvement or bug, but launch ships fine without it (Linear calls this "Normal", priority value 3)
- **Low** — nice-to-have, nitpick, backlog-forever candidate

### Description body

Keep bodies lightweight. Never pad.

**Always include:**
- 1-2 line context (why this matters or what's happening)

**For features / improvements, also include:**
- 1-2 lines on "what done looks like" (e.g. "we need this to be X")

**For bugs, also include:**
- Numbered repro steps
- Add expected vs actual only when the bug is non-obvious from title + steps

**If a screenshot was captured:**
- Add a line at the bottom: `[screenshot: /tmp/shot-xxx.png — attach manually]`

### Example tickets

**Bug:**
```
Title: widget: voice note save button misaligned on mobile
Label: Bug
Priority: Normal
Project: (match if exists)

Body:
Save button overlaps the close icon on screens < 380px wide — users tap close by accident.

Repro:
1. Open widget on iPhone SE
2. Start voice note
3. Try to tap save

[screenshot: /tmp/shot-a12.png — attach manually]
```

**Feature:**
```
Title: integrations: add discord webhook support
Label: Feature
Priority: Low
Project: integrations

Body:
Users have asked for discord routing alongside slack.
We need this to send the same payload shape as the slack integration, configured via a webhook URL in settings.
```

**Improvement:**
```
Title: capture: faster area selector on large screens
Label: Improvement
Priority: Normal
Project: (match if exists)

Body:
Area selector lags noticeably on 4K monitors during drag.
We need this to stay smooth (60fps) regardless of viewport size.
```

## Linear project format

When creating Linear projects for this repo, follow this spec exactly. Projects are an orthogonal dimension to ticket area prefixes: prefixes organize tickets by area (forever), projects wrap finite initiatives by outcome (until they ship).

### When to create a project

Claude senses project-worthy work on 4 signals:

1. **Named shippable outcome** — a ticket or conversation references "launch", "ship", "v0.X", "release", or a similar shippable milestone
2. **Cluster** — 3+ related tickets exist that share a single outcome AND are not already attached to an existing project
3. **Cross-area sweep** — one outcome touches 2+ title prefix areas (e.g. `launch` spans `widget` + `server` + `docs` + `infra`)
4. **Explicit request** — Parth or collaborator says the word "project" or "initiative"

Signals 1 and 4 can fire from a single ticket. Signals 2 and 3 require multiples.

### Who writes the project (hybrid)

- **Signals 1 & 4** (human initiates): Parth or collaborator writes the outcome statement and description body. Claude asks for them before creating.
- **Signals 2 & 3** (Claude detects): Claude drafts name + outcome + description + ticket list, then **proposes to Parth and waits for approval** before creating anything in Linear. Never auto-create on detection.

### Fields (always set)

| Field | Rule |
|---|---|
| Name | lowercase, descriptive, short, no schema or prefix (e.g. `launch v0.1`, `docs site`, `github integration rollout`) |
| Outcome statement | required, format: `done when: X` where X is concrete and verifiable — the linchpin of closure |
| Description body | 3-line framework, bolded labels, line breaks between (see below) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fvckprth/remediate](https://github.com/fvckprth/remediate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
