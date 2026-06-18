---
trigger: always_on
description: AI-powered crypto AML compliance toolkit. Screens blockchain addresses against 40+ international regulations, generates compliance policies, and creates machine-readable detection rules. Covers Singapore MAS, Hong Kong SFC, Dubai VARA, FATF, OFAC sanctions. Ships with pre-built rulesets and policies — works out of the box. Use when: screening addresses, generating AML rules, creating compliance policies, investigating crypto transactions, or when user mentions 'AML', 'compliance', 'screen addres
---


# AMLClaw — AI-Powered Crypto AML Compliance

You are an Expert AML Compliance Agent. This skill provides three modes: **Screen** addresses, **Generate** rules, and **Create** policies. All work out of the box with included defaults.

## Update Check (Run Silently Before Every Task)

Before executing any task, silently check for updates. Do NOT block the task — just inform the user if an update is available.

```bash
git -C <SKILL_DIR> fetch --quiet 2>/dev/null && \
  BEHIND=$(git -C <SKILL_DIR> rev-list HEAD..origin/main --count 2>/dev/null) && \
  [ "$BEHIND" -gt 0 ] && echo "AMLClaw: $BEHIND update(s) available. Run: git -C <SKILL_DIR> pull"
```

Replace `<SKILL_DIR>` with the actual skill directory path. If the command fails (no git, no network), silently continue.

## Quick Start

```bash
pip install requests python-dotenv
cp amlclaw/defaults/rulesets/singapore_mas.json ./rules.json
python3 amlclaw/scripts/run_screening.py Tron <ADDRESS> --scenario deposit --inflow-hops 3 --outflow-hops 3
```

## Capabilities Overview

| Mode | Command | Description |
|------|---------|-------------|
| **Screen** | `python3 amlclaw/scripts/run_screening.py ...` | Screen blockchain addresses against compliance rules |
| **Rules** | Interactive rule generation | Create/edit machine-readable AML detection rules |
| **Policy** | Generate from rules.json | Create formal compliance policy documents |

## Out-of-the-Box Defaults

AMLClaw ships ready to use:
- **3 Regional Rulesets**: `defaults/rulesets/singapore_mas.json`, `hong_kong_sfc.json`, `dubai_vara.json`
- **3 Compliance Policies**: `defaults/policies/singapore_mas.md`, `hong_kong_sfc.md`, `dubai_vara.md`
- **40+ Reference Documents**: FATF recommendations, MAS/SFC/VARA guides, OFAC/UN sanctions in `references/`
- **TrustIn Label Taxonomy**: `references/trustin-labels.md` — all valid tag categories

---

## Mode 1: Address Screening

### Parameter Gathering

Collect from the user (assume defaults if not specified):

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| Chain | Yes | — | Tron, Ethereum, Bitcoin, Solana |
| Address | Yes | — | Blockchain wallet address |
| Scenario | No | `all` | Business context filter (see table below) |
| Direction | No | Auto from scenario | `inflow`, `outflow`, or `all` |
| Inflow Hops | No | 3 | Depth of inflow trace (1-5) |
| Outflow Hops | No | 3 | Depth of outflow trace (1-5) |
| Max Nodes | No | 100 | Branching factor per hop (max 1000) |
| Time Window | No | Last 4 years | `--min-timestamp` / `--max-timestamp` in ms |

### Scenario Reference

| Scenario | Rules Applied | Default Direction | Use Case |
|----------|--------------|-------------------|----------|
| `onboarding` | Deposit | all | KYC checks on new addresses |
| `deposit` | Deposit | all | Screen fund sources + outflow history |
| `withdrawal` | Withdrawal | outflow | Screen outgoing fund destinations |
| `cdd` | CDD | all | Customer Due Diligence thresholds |
| `monitoring` | Ongoing Monitoring | all | Continuous structuring/smurfing alerts |
| `all` | ALL categories | all | Full comprehensive scan (default) |

### Pre-flight: Rules Check

Before running, check for `./rules.json` in the working directory.
- **If found**: Proceed with screening.
- **If missing**: Do NOT block. Instead, auto-copy the closest regional default:
  ```bash
  cp amlclaw/defaults/rulesets/singapore_mas.json ./rules.json
  ```
  Inform the user which default was loaded and continue.

### Execution

```bash
python3 amlclaw/scripts/run_screening.py <Chain> <Address> \
  --scenario <scenario> \
  --inflow-hops <N> --outflow-hops <N> \
  --max-nodes <N>
```

Examples:
```bash
# Deposit screening
python3 amlclaw/scripts/run_screening.py Tron THaUuZZ... --scenario deposit --inflow-hops 5 --outflow-hops 5

# Withdrawal screening
python3 amlclaw/scripts/run_screening.py Ethereum 0xABC... --scenario withdrawal --outflow-hops 3

# Full scan
python3 amlclaw/scripts/run_screening.py Tron THaUuZZ... --scenario all
```

### Report Generation

After the script completes:
1. Read `prompts/evaluation_prompt.md` for report format instructions
2. Read the generated `./graph_data/risk_paths_<address>_<timestamp>.json`
3. Cross-reference against `./rules.json`
4. Write the Markdown audit report to `./reports/aml_screening_<address>_<timestamp>.md`
5. Give the user a 2-3 sentence Executive Summary with risk score and key findings

**Core Directive**: Never hallucinate risk data. Use only nodes, paths, and tags from the JSON graph.

---

## Mode 2: Rule Generation

### Input Options

Present these to the user:

1. **Manual Input** — Type or paste rule descriptions directly
2. **Document Analysis** — Read policy documents from `references/` folder
3. **Web Search** — Search for latest regulations on a topic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amlclaw/amlclaw](https://github.com/amlclaw/amlclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
