---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Repository Overview

`sap-engineering-skill` is a monorepo of AI agent skills for SAP ABAP engineering. Each skill follows the `SKILL.md` specification and works with compatible AI agent frameworks (opencode, Claude Code, Cursor).

### Skills

| Skill | Purpose | Location |
|-------|---------|----------|
| `sap-adt-cli` | Read/write ABAP source and metadata via ADT REST API | `skills/sap-adt-cli/` |
| `abap-code-review` | Pre-release security & quality review (9 dimensions) | `skills/abap-code-review/` |
| `sap-transport-gate` | Transport Request release gate assessment (10 dimensions) | `skills/sap-transport-gate/` |
| `sap-integration-wiki` | SAP integration knowledge base (9 domains, 8 technologies) | `skills/sap-integration-wiki/` |

The three subtree skills (`abap-code-review`, `sap-transport-gate`, `sap-integration-wiki`) are maintained as independent public repositories and are tracked via git subtree remotes.

---

## Repository Structure

```
sap-engineering-skill/
├── skills/
│   ├── sap-adt-cli/              ← Source in this repo (main skill)
│   │   ├── scripts/
│   │   │   ├── sap_adt_cli.py    ← Main CLI entry point
│   │   │   └── lib/              ← config.py, handlers.py, client.py
│   │   └── SKILL.md              ← Skill spec for agent frameworks
│   ├── abap-code-review/         ← [git subtree] independent repo
│   │   ├── references/           ← REF_ABAP_SECURITY.md, REF_CLEAN_ABAP.md
│   │   │                        └── REPORT_TEMPLATE.md
│   │   └── SKILL.md
│   ├── sap-transport-gate/       ← [git subtree] independent repo
│   │   ├── scripts/
│   │   │   ├── tr_collector.py   ← Online TR collection CLI
│   │   │   └── lib/              ← config.py, handlers.py, client.py
│   │   ├── references/           ← Decision policy, review dimensions, etc.
│   │   ├── evals/                ← Golden set, evals.json
│   │   └── SKILL.md
│   └── sap-integration-wiki/     ← [git subtree] independent repo
│       ├── references/           ├── scenarios/, tech/, troubleshoot/
│       ├── scripts/              ← gen-odata-postman.js, gen-jco-config.py, etc.
│       ├── assets/               ├── payloads/, configs/
│       └── SKILL.md
├── README.md
├── README.zh-CN.md
├── LICENSE
└── setup-opencode-abap-cli.bat   ← Windows one-click installer
```

---

## Git Subtree Management

Three skills are tracked as independent subtrees from public repositories:

| Remote | Repository | Subtree Path |
|--------|-----------|--------------|
| `pub-abap-code-review` | https://github.com/shrek-abaper/abap-code-review | `skills/abap-code-review/` |
| `pub-sap-transport-gate` | https://github.com/shrek-abaper/sap-transport-gate | `skills/sap-transport-gate/` |
| `pub-sap-integration-wiki` | https://github.com/shrek-abaper/sap-integration-wiki | `skills/sap-integration-wiki/` |

**Pull changes from subtree:**
```bash
git subtree pull --prefix=skills/abap-code-review pub-abap-code-review main --squash
git subtree pull --prefix=skills/sap-transport-gate pub-sap-transport-gate main --squash
git subtree pull --prefix=skills/sap-integration-wiki pub-sap-integration-wiki main --squash
```

**Push changes to subtree:**
```bash
git subtree push --prefix=skills/abap-code-review pub-abap-code-review main
```

When working within a subtree skill, changes should eventually be pushed to the corresponding public repository.

---

## Common Commands

### sap-adt-cli (ABAP CLI Tool)

The CLI entry point is `scripts/sap_adt_cli.py` inside the skill directory.

**Configure SAP credentials:**
```bash
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py configure
```

**Check connection status:**
```bash
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py status
```

**Read ABAP objects (examples):**
```bash
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py get-program SAPMV45A
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py get-class ZCL_MY_CLASS
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py get-table VBAK
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py get-package ZMYPACKAGE
```

**Write operations (requires `allow_write: true` + confirmation):**
```bash
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py write-source class ZCL_MY_CLASS --file /tmp/zcl.abap
python3 skills/sap-adt-cli/scripts/sap_adt_cli.py activate class ZCL_MY_CLASS
```

### sap-transport-gate (TR Collection CLI)

**Configure SAP credentials:**
```bash
python3 skills/sap-transport-gate/scripts/tr_collector.py configure
```

**Collect transport package:**
```bash
python3 skills/sap-transport-gate/scripts/tr_collector.py collect DEVK900123 --output-dir reports/DEVK900123_package/ --verbose
```

### Testing

Each skill's evals are stored in `evals/` directories. Run evals using the skill creator's evaluation framework or the JSON-based eval definitions in `evals/evals.json`.

---

## Architecture Patterns

### CLI Tools (sap-adt-cli & sap-transport-gate)

Both Python CLI tools share a common `lib/` pattern:

```
scripts/
├── <entrypoint>.py          ← Main Click CLI
└── lib/
    ├── __init__.py
    ├── config.py            ← Config file loading, credential management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shrek-abaper/sap-engineering-skill](https://github.com/shrek-abaper/sap-engineering-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
