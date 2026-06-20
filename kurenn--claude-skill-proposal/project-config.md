---
trigger: always_on
description: Generate beautiful single-page client proposals as a document-format replacement for PDFs and Google Docs. Persists answers, fetches brand colors from the client's site (with WCAG contrast warnings), generates a Tailwind-built single-page HTML, and runs the `/critique` design skill via the Skill tool to grade and fix the output. Includes dynamic OG images for unfurls and version-history snapshots on revise. Use when the user says "create a proposal", "client proposal", "scope of work", "SOW", or
---


# /proposal

Generate single-page client proposals as a beautiful, branded HTML document — the replacement for PDF / Google Doc proposals. Static, no live actions, just a polished read.

## Operating principle

A proposal does not close a deal — the conversation does. The proposal is the artifact your champion hands to people who weren't in the room. Its job is to **reduce career risk for the champion** by giving them ammunition to defend the choice. Every section earns its place by pre-empting an objection the champion will face.

This skill is **document-replacement**, not a SaaS product. There are no live buttons, no accept tracking, no view analytics. The output is a beautifully formatted page deployed to a URL the seller emails or pastes in Slack. The buyer reads it the way they'd read a PDF and replies through the existing email thread.

If the seller can't answer the discovery questions in the buyer's own words, the proposal won't save the deal. Push back. Don't generate from a thin brief.

## Skill assets (in `~/.claude/skills/proposal/`)

```
SKILL.md                          ← you are here
template.html                     ← the single-page proposal template
tailwind-input.css                ← Tailwind source for precompiled CSS
tailwind.config.js                ← content paths for JIT
discovery-schema.json             ← canonical schema for discovery.json
extract-colors.sh                 ← multi-source brand color extractor (WCAG-aware)
seller-defaults.json              ← persisted seller info — auto-loaded if present
examples/
  ├── saas-acme.html              ← reference: SaaS implementation pattern
  ├── agency-globex.html          ← reference: agency/creative pattern
  └── services-initech.html       ← reference: professional services pattern
vercel-starter/
  ├── vercel.json                 ← clean URLs + security headers + asset caching
  ├── index.html                  ← private root landing page
  ├── package.json                ← @vercel/og dependency for OG image route
  ├── assets/tailwind.css         ← precompiled, production-ready
  └── api/
      └── og.tsx                  ← dynamic OG image route (only runtime asset)
```

**Read `examples/*.html` before generating.** They are the canonical reference for what a good filled-in proposal looks like across industries.

## When to use / when NOT to

- ✅ Sending a proposal, scope of work, SOW, quote, or pitch document
- ✅ User has had a discovery call and needs to put it in writing
- ❌ Marketing landing page → `/copy-board` or `/prototype`
- ❌ Internal planning docs → write directly
- ❌ Multi-page deck → this skill is single-page only by design
- ❌ User has not had any conversation with the prospect → tell them to have one first

## Argument routing

| Invocation | Behavior |
|---|---|
| `/proposal` | Full new-proposal flow (Steps 0–7) |
| `/proposal Acme Corp` | Same, with client name pre-filled |
| `/proposal --revise <slug>` | Load `proposals/<slug>/discovery.json`, ask what's changed, snapshot current to `versions/v{n}.html`, regenerate, redeploy |
| `/proposal --clone <slug>` | Load `proposals/<slug>/discovery.json` as a starting point, ask what's different, save to a NEW slug |
| `/proposal --meetings <src>[,<src>...]` | New-proposal flow with discovery answers drafted from one or more transcripts. Each chunk presents the draft and waits for seller confirmation/edits before saving. |
| `/proposal "Acme Corp" --meetings <src>,<src>` | Same as above, with client name pre-filled. |
| `/proposal --revise <slug> --meetings <src>,...` | Revise existing proposal using new meeting context. Diffs the saved discovery against extracted transcript answers and asks the seller which sections to update. |

**`<src>` accepts**:
- **Circleback URL** — e.g., `https://app.circleback.ai/meetings/<id>`. Resolved via the Circleback MCP connector (`ReadMeetings` + `GetTranscriptsForMeetings`). The connector must be authorized in the user's environment.
- **Local file path** — `.txt`, `.md`, `.vtt`, `.srt`, or `.json` (Circleback export format). Use this for offline notes or transcripts from other tools.

**Combinability rules**:
- `--meetings` IS allowed with: a positional client name, `--revise <slug>`.
- `--meetings` is NOT allowed with `--clone`. If both are passed, abort with: *"`--clone` and `--meetings` are not compatible. Use `--meetings` with `--revise` to layer new context onto an existing proposal, or omit `--clone` to start fresh from transcripts."*

## The flow

```
0. Bootstrap working dir         → scaffold vercel.json, assets, api/og.tsx
1. Load seller-defaults.json     → auto-fill seller's company info (skip questions)
1.5 Transcript ingestion         → (only if --meetings) fetch + normalize sources,
                                   pre-extract candidate answers w/ provenance,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kurenn/claude-skill-proposal](https://github.com/kurenn/claude-skill-proposal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
