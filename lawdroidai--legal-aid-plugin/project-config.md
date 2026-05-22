---
trigger: always_on
description: CONFIGURATION LOCATION
---

<!--
CONFIGURATION LOCATION

User-specific configuration for this plugin lives at a version-independent path that survives plugin updates:

  ~/.claude/plugins/config/claude-for-legal/legal-aid/CLAUDE.md

Rules for every skill, command, and agent in this plugin:
1. READ configuration from that path. Not from this file.
2. If that file does not exist or still contains [PLACEHOLDER] markers, STOP before doing substantive work. Say: "This plugin needs setup before it can give you useful output. Run /legal-aid:cold-start-interview — it takes about 15-20 minutes and every command in this plugin depends on it. Without it, outputs will be generic and may not match how your office actually works." Do NOT proceed with placeholder or default configuration. The only skills that run without setup are /legal-aid:cold-start-interview itself and any --check-integrations flag.
3. Setup and cold-start-interview WRITE to that path, creating parent directories as needed.
4. On first run after a plugin update, if a populated CLAUDE.md exists at the old cache path
   (~/.claude/plugins/cache/claude-for-legal/legal-aid/<version>/CLAUDE.md for any version)
   but not at the config path, copy it forward to the config path before proceeding.
5. This file (the one you are reading) is the TEMPLATE. It ships with the plugin and shows the structure the config should have. It is replaced on every plugin update. Never write user data here.

**Shared company profile.** Office-level facts (who you are, what you do, where you operate, funding sources, key people) live in `~/.claude/plugins/config/claude-for-legal/company-profile.md` — one level above this file, shared by all plugins in the suite. Read it before this plugin's practice profile.
-->

# Civil Legal Aid Practice Profile

*Written by the managing-attorney-facing cold-start interview. Staff don't edit this — they run `/onboard`. If you see `[PLACEHOLDER]` below, run `/legal-aid:cold-start-interview`.*

---

## Who's using this

**Role:** [PLACEHOLDER — Managing attorney / Executive Director (default, required to run setup) | Staff attorney | Paralegal | Intake specialist | Volunteer attorney]

Setup must be run by a managing attorney or person with equivalent authority over the office's intake and supervision practice. Other staff onboard via `/legal-aid:onboard`. Clients are not plugin users — they are the people the office serves, and their materials flow through staff and attorney outputs rather than through direct plugin use.

**Managing attorney(s):** [PLACEHOLDER — name(s), bar admission jurisdiction(s), bar number(s)]
**Supervisory authority basis:** [PLACEHOLDER — e.g., "ED + Director of Litigation; managing attorney for each practice unit"]
**Ethical preconditions confirmed:** [PLACEHOLDER — yes / no; list unresolved items if any. Captured from Part 0 ethical preconditions.]
**Funder restrictions acknowledged:** [PLACEHOLDER — list each funding source whose restrictions the managing attorney has confirmed staff understand: LSC 45 CFR Part 1600, HUD, VAWA, state IOLTA, specific foundation grants, etc.]

When the role is managing attorney, staff attorney, paralegal, or intake specialist, every output this plugin produces is supervised staff work. The AI-assisted draft label is the canonical header for staff outputs in this environment — it replaces a generic privilege / non-lawyer notice.

**Consequential-action note:** Sending a client letter, filing with a court or agency, closing a case, and making representation decisions are gated by the office's supervision workflow (see `## Supervision style` below). The Part 0 role check reinforces that gate. Do not bypass the supervision workflow even when the plugin's internal checks pass.

---

## Available integrations

| Integration | Status | Fallback if unavailable |
|---|---|---|
| Case management system (Legal Server / LegalFiles / Clio / PIKA / JusticeServer) | [✓ / ✗ / which one] | Case metadata captured in local intake / status files; no auto-sync |
| Document storage (Google Drive / SharePoint / Box) | [✓ / ✗] | Staff outputs save to local filesystem; review stays in-plugin |
| Intake-routing connector (provider-specific, optional) | [✓ / ✗ / which one] | `/eligibility-screening` and `/client-intake` use built-in templates without external triage signal |
| Procedural-retrieval connector (provider-specific, optional) | [✓ / ✗ / which one] | Citations tagged `[model knowledge — verify]` rather than `[provider — pinned]` |

*Re-check: `/legal-aid:cold-start-interview --check-integrations`*

---

## Office profile

**Office:** [PLACEHOLDER — name] *(From company-profile.md)*
**Type:** [PLACEHOLDER — LSC-funded legal services / non-LSC civil legal aid / hybrid / court self-help center / public defender civil division / pro bono coordinator office]
**Practice areas:** [PLACEHOLDER — housing / family / consumer / immigration / public benefits / civil rights / elder / DV / other] *(From company-profile.md)*
**Funding sources:** [PLACEHOLDER — LSC / IOLTA / HUD / VAWA / VOCA / state appropriation / foundation / private]
**Staff size:** [PLACEHOLDER — attorneys / paralegals / intake specialists]
**Typical annual intake volume:** [PLACEHOLDER]
**Typical active caseload per attorney:** [PLACEHOLDER]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lawdroidAI/legal-aid-plugin](https://github.com/lawdroidAI/legal-aid-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
