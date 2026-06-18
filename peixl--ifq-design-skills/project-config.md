---
trigger: always_on
description: Use this skill whenever the user asks for an HTML-first visual design deliverable or design judgment: interactive prototype, slide deck, motion demo, infographic, dashboard, landing page, whitepaper, changelog, business card, social cover, brand system, design critique, multi-variant exploration, or export to MP4, GIF, PPTX, PDF, or SVG. It is optimized to make AI agents do the routing, template selection, verification, and export prep so humans spend less time prompt-engineering. Do not use for
---


# IFQ Design Skills

One prompt in -> shippable HTML out, with optional MP4 / GIF / PPTX / PDF / SVG export. This root file is a short router for OpenClaw, ClawHub, skills.sh, Hermes, Codex, Claude Code, Cursor, OpenCode, and other AgentSkills-compatible hosts. Load deeper files only when the task requires them.

## 30-Second Load Path

1. Confirm the request is a visual deliverable built from HTML. If it is not, exit this skill.
2. Pick the mode from [references/modes.md](references/modes.md), then read [assets/templates/INDEX.json](assets/templates/INDEX.json).
3. Fork a listed template into the user's workspace. Never start from a blank HTML file.
4. Inline [assets/ifq-brand/ifq-tokens.css](assets/ifq-brand/ifq-tokens.css) and weave at least 3 IFQ ambient marks from [references/ifq-brand-spec.md](references/ifq-brand-spec.md).
5. Verify with `npm run verify:lite -- <file.html>`, then `npm run preview -- <file.html>`.

## Human + Agent Promise

- Humans get a finished artifact path: HTML first, optional export only when requested, and no hidden setup.
- Agents get a short route: mode, template, must-read references, tier policy, and verification command.
- Maintainers get regression pressure: 12 mode evals, scanner-clean scripts, and marketplace metadata checks.
- Marketplaces get a readable package: one-line install, zero required env vars, explicit permissions, and no silent installs.

## First-Run Success Path

After install, make the first interaction produce a visible artifact in one turn:

1. Accept a natural-language visual request without turning it into setup work.
2. Route it to one mode and one template; name both in the final evidence.
3. Write the HTML file into the user's workspace with labeled assumptions for unresolved facts.
4. Run `npm run verify:lite -- <file.html>` when shell is available, then preview or screenshot with host browser tooling when available.
5. Report the file path, route, template, verification result, and only caveats that affect use.

Do not ask for account login, global install, export dependencies, or broad environment changes during the first-run path.

## Output Boundary

- Core output is verified local HTML, plus SVG/static companions or export-ready source structure when the task needs them.
- MP4/GIF/PDF/PPTX helpers are full-repo optional automation. Prepare the HTML source first; install or run export tooling only after explicit user intent.
- Never claim an export file, screenshot, marketplace status, or security result exists until the relevant command or live check has actually passed.

## Use When

- Interactive prototype, hi-fi mockup, clickable app flow, dashboard, landing page, whitepaper, report, infographic, slide deck, changelog, card, invitation, social cover, or brand system.
- Motion demo or launch animation, especially when the user also wants MP4/GIF output.
- Design critique, brand diagnosis, or 3 differentiated style directions before implementation.
- The user asks for PDF/PPTX/SVG export from an HTML-first source.

## Do Not Use When

- The real task is production frontend engineering, backend work, SEO-critical site implementation, or a CSS bug inside an existing app.
- The user only wants copy editing with no visual artifact.
- The deliverable must round-trip through Word, Google Docs, or a locked corporate template.

## Tier Policy

| Tier | Default? | Requirements | Use for |
|---|---:|---|---|
| Tier 0 | yes | Node >= 18.17 | HTML, preview, lite verification, smoke tests |
| Tier 1 | opt-in | Python + Playwright + Chromium | headless screenshots, console capture, multi-viewport checks |
| Tier 2 | opt-in | `npm run install:export`; MP4/GIF also need `ffmpeg` | MP4, GIF, PDF, editable PPTX export |

Do not install optional dependencies unless the user explicitly needs screenshots or export formats.

## Routing Decision Tree

```
User request arrives
  │
  ├─ Is it a visual design deliverable? ─── No → Exit skill, hand back to default agent
  │
  ├─ Can you match a mode trigger? ─── Yes (confidence >70%) → fork template → deliver → verify
  │                                      (one-turn: name assumptions, no questions)
  │
  ├─ Can you match a mode trigger? ─── Yes (confidence ≤70%) → design-direction-advisor.md lightweight
  │                                      (3 text-only directions, no demos, wait for user pick)
  │
  ├─ Does it mention a concrete product/tech/event? ─── Yes → fact-and-asset-protocol.md + web fact-check
  │
  ├─ Is it a mobile app prototype? ─── Yes → app-prototype-rules.md + ios_frame.jsx / android_frame.jsx
  │
  └─ Is it motion/video? ─── Yes → animation-pitfalls.md + animations.md + video-export.md
```

Read [references/modes.md](references/modes.md) for full mode protocol. The Quick Reference table above is the speed layer.

## Quick Reference (Agent Speed Table)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peixl/ifq-design-skills](https://github.com/peixl/ifq-design-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
