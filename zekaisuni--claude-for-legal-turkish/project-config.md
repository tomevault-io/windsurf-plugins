---
trigger: always_on
description: CONFIGURATION LOCATION
---

<!--
CONFIGURATION LOCATION

User-specific configuration for this plugin lives at a version-independent path that survives plugin updates:

  ~/.claude/plugins/config/claude-for-legal/ai-governance-legal/CLAUDE.md

Rules for every skill, command, and agent in this plugin:
1. READ configuration from that path. Not from this file.
2. If that file does not exist or still contains [PLACEHOLDER] markers, STOP before doing substantive work. Say: "This plugin needs setup before it can give you useful output. Run /ai-governance-legal:cold-start-interview — it takes about 10-15 minutes and every command in this plugin depends on it. Without it, outputs will be generic and may not match how your practice actually works." Do NOT proceed with placeholder or default configuration. The only skills that run without setup are /ai-governance-legal:cold-start-interview itself and any --check-integrations flag.
3. Setup and cold-start-interview WRITE to that path, creating parent directories as needed.
4. On first run after a plugin update, if a populated CLAUDE.md exists at the old cache path
   (~/.claude/plugins/cache/claude-for-legal/ai-governance-legal/<version>/CLAUDE.md for any version)
   but not at the config path, copy it forward to the config path before proceeding.
5. This file (the one you are reading) is the TEMPLATE. It ships with the plugin and shows the
   structure the config should have. It is replaced on every plugin update. Never write user data here.

**Shared company profile.** Company-level facts (who you are, what you do, where you operate, your risk posture, key people) live in `~/.claude/plugins/config/claude-for-legal/company-profile.md` — one level above this file, shared by all 12 plugins. Read it before this plugin's practice profile. If it doesn't exist, this plugin's setup will create it.
-->

# AI Governance Practice Profile

*Written by the cold-start interview. Until then, this is a template — if you see
`[PLACEHOLDER]`, run `/ai-governance-legal:cold-start-interview`.*

---

## Company profile

[Company] is a [description — what the company does and who its customers are]. *(From company-profile.md — edit there to change across all plugins)*

**AI role:** *Not set at company level.* Under the EU AI Act, role (provider,
deployer, importer, distributor, authorized representative, product
manufacturer) is assessed **per AI system** — see `## AI system inventory`
below. A single organization can be a provider of one system and a deployer
of another; a single company-level label produces wrong answers.

**AI activity summary:** [PLACEHOLDER — one-paragraph sketch of how AI touches
the company overall: whether you build, deploy, consume vendor AI, train
models, or some mix. This is orientation only. The authoritative per-system
classification lives in `ai-systems.yaml`.]

**Regulatory footprint:** [PLACEHOLDER — only list what actually applies.
EU AI Act / Colorado / BIPA / sector-specific / contractual requirements only.
If nothing applies yet, say so.] *(From company-profile.md — edit there to change across all plugins)*

**Open regulatory matters:** [PLACEHOLDER]

**External commitments:** [PLACEHOLDER — voluntary AI commitments, public AI
principles page, transparency reports — or none]

**Practice setting:** [PLACEHOLDER — Solo/small firm | Midsize/large firm | In-house | Government/legal aid/clinic] *(From company-profile.md — edit there to change across all plugins)*

---

## Who's using this

**Role:** [PLACEHOLDER — Lawyer / legal professional | Non-lawyer with attorney access | Non-lawyer without attorney access]
**Attorney contact:** [PLACEHOLDER — name / team / outside firm / N/A]

---

## Available integrations

| Integration | Status | Fallback if unavailable |
|---|---|---|
| Document storage (Google Drive / SharePoint / Box) | [✓ / ✗] | Manual file paths; outputs land locally |
| Scheduled-tasks | [✓ / ✗] | Policy-monitor sweep runs on demand only |
| Slack | [✓ / ✗] | Escalations and notifications go via email only |

*Re-check: `/ai-governance-legal:cold-start-interview --check-integrations`*

---

## Use case registry

*Extracted from interview. Add new use cases as they arise.*

| Use case | Approved | Conditions / Requirements | Never — reason |
|---|---|---|---|
| [PLACEHOLDER] | | | |

### Red lines

The following are automatic nos, regardless of how a request is framed:

- [PLACEHOLDER — red line 1 and reason]
- [PLACEHOLDER — red line 2 and reason]

### Governance tiers

| Risk tier | Approval path | Example use cases |
|---|---|---|
| Standard | [PLACEHOLDER] | Internal productivity tools, assistive drafting |
| Elevated | [PLACEHOLDER — legal / privacy review required] | Customer-facing AI, HR use cases |
| High | [PLACEHOLDER — C-suite or board] | Consequential automated decisions, biometric |

---

## AI system inventory

**Inventory file:** `~/.claude/plugins/config/claude-for-legal/ai-governance-legal/ai-systems.yaml`

Under the EU AI Act, **role and risk tier are assessed per AI system, not per
company.** A single organization can be a provider of System A, a deployer of
System B, and an importer of System C — each combination triggers a different
set of obligations. This inventory stores one record per system.

Each record carries:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZekaiSuni/claude-for-legal-turkish](https://github.com/ZekaiSuni/claude-for-legal-turkish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
