---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CLAUDE.md — Project Rules

> Copy this whole folder for each new project, then fill in the placeholders below. This file tells Claude how to work on **this** project. It pairs with `project-context.md` (the values) and the installed OutSystems skills (the behavior).

## Project

- **Customer:** `World Bank Group`
- **Project:** `WBG`
- **Environment:** ODC (default) — change if O11
- **Class prefix:** `loop-`
- **Spacing base:** unconfirmed — grid spec TBD (do **not** flag values as "off the 4pt grid"). Scale extracted from Figma (`tokens/spacing.css`, descriptive `tiny`…`xhuge`); most steps are multiples of 4 but this is not a confirmed rule. See the 2026-06-16 Live Style Guide sync ("final grid spec not yet confirmed").
- **Token style:** OutSystems UI standard conventions
- **CSS methodology:** BEM (`block__element--modifier`)
- **Custom components:** vanilla JS Web Components wrapped in OutSystems Blocks
- **Accessibility target:** WCAG 2.2 AA — **fidelity-first**

## The one rule that matters most here

**Build the design exactly as specified. Never silently change a brand color, value, or token to satisfy accessibility or to "tidy" the design.** When the design conflicts with accessibility, brand, or token rules, implement it faithfully and raise a **finding** (see `findings/`). The finding carries the recommendation back to design; the code stays true to the mockup until design responds or the brand owner signs off.

Implementation-level accessibility that does NOT change the visual design — focus rings in the design's own colors, keyboard handlers, ARIA, semantic HTML, reduced-motion, labels — is applied automatically.

## Findings routing (used by the `outsystems-design-findings` skill)

```
findings.ticketing     = github            # default; alternatives: notion | jira
findings.ticket_target = <owner/repo>       # + optional GitHub Project name
findings.slack_channel = <#channel | none>  # GitHub Slack app or Slack connector
findings.gate          = high+              # high+ opens issues + notifies; medium/low batch to the register
```

Findings become **GitHub Issues filed as Bugs** (Bug issue type + `bug` label) in the project repo, created via `gh` from Claude Code (works on the private repo, no MCP). Labels: `finding` + `bug` + type (`a11y`/`brand`/`token`/`consistency`) + `sev:*`. A structured issue form lives at `.github/ISSUE_TEMPLATE/finding.yml`. The local register mirror is `findings/findings-register.md`; payloads are written to `findings/tickets/`.

## Code handover (this dev works mainly in OutSystems)

Generated code (CSS, Web Component `.js`, Block instructions) is **not** the end of the chat — it is handed over as a **GitHub issue filed as a Task and assigned to the developer**, so they add it into OutSystems themselves. Label `handover` + `task`; form at `.github/ISSUE_TEMPLATE/handover.yml`; example body in `handover/`.

**Rule: the handover ticket must CONTAIN the JS/CSS to copy into ODC — not just point at a repo path.** Every `handover/*.md` carries a `## Code to paste into ODC` section with the verbatim artifact(s) in a collapsed `<details>` block (source path in the `<summary>`). Tokens travel via `dist/tokens.css` (its own paste) so they aren't duplicated there. Embed only what the developer hand-places: block CSS overrides and Web Component `.js`. The blocks are generated from source by `node build/embed-handover-code.mjs` (idempotent) — re-run it after editing a source file, and add new handovers to its `MAP`.

**Rule: every handover also carries a `## Build in ODC with Mentor Studio` section** — a ready-to-paste prompt for **ODC Mentor Studio** that scaffolds the OutSystems side (Block, attribute bindings, event wiring, Client Actions). Mentor is a logic/data agent — it does *not* author the CSS or the Web Component, so the prompt fences those off as already-pasted and aims Mentor only at the wiring. The same `embed-handover-code.mjs` generates it: archetype-aware by default (Web Component / native-widget restyle / Style-Guide reference), or fully filled when the `MAP` entry supplies a `mentor` spec (see `loop-toast.md` for the worked example). The reusable template lives at `handover/MENTOR-STUDIO-PROMPT.md`.

```bash
gh issue create --title "[handover] <component> — add in OutSystems" \
  --body-file handover/<artifact>.md --label "handover,task" --type "Task" \
  --assignee @me --repo <owner/repo>
```

Both findings (bugs) and handovers (tasks) can live on the same GitHub **Project** board — a kanban view with a `Status` column you drag items across.

## Build pipeline

- Source tokens live in `tokens/` (`colors.css`, `spacing.css`, `typography.css`).
- `npm run build:theme` → **two ODC pastes** (the 2026-07-16 token split):
  **`dist/tokens.css`** — the design tokens ONLY (single consolidated `:root` +
  device-scoped token redefinitions like `body.phone` type steps) — and
  **`dist/theme.css`** — everything else (`@font-face`, base rules, utility
  classes, widget/component overrides; NO tokens). Both pasted into the ODC
  Theme editor; token edits only require re-pasting `dist/tokens.css`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kharloridado/wbg](https://github.com/kharloridado/wbg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
