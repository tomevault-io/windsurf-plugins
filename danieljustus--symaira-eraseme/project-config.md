---
trigger: always_on
description: [PB-2026-09-09](docs/product-boundaries.md) keeps EraseMe an independent specialized product. Browse and Operate become optional Brain modules; the credential service remains independently usable. None becomes a mandatory Brain gateway/context dependency for EraseMe. Existing command/API, privacy, evidence and approval contracts remain unchanged until explicit tested migrations. Do not couple privacy workflows to the presence of Brain's GUI.
---

# AI Agent Integration Guide

## Current product contract

[PB-2026-09-09](docs/product-boundaries.md) keeps EraseMe an independent specialized product. Browse and Operate become optional Brain modules; the credential service remains independently usable. None becomes a mandatory Brain gateway/context dependency for EraseMe. Existing command/API, privacy, evidence and approval contracts remain unchanged until explicit tested migrations. Do not couple privacy workflows to the presence of Brain's GUI.

**Symaira EraseMe** supports all major AI coding agents through standardized
skill formats and adapter files.

## Ecosystem Guidance

- Before changing cross-tool integrations, shared conventions, or product
  boundaries, read [PB-2026-09-09](docs/product-boundaries.md); it is available in standalone checkouts.
- Keep the standalone-first contract: this repo must install, test, and run
  without any other Symaira tool installed.

## Quick Reference

| Agent | Format | Auto-Discovery | Setup Complexity |
|-------|--------|----------------|------------------|
| [Claude Code](#claude-code) | `SKILL.md` | ✅ `.claude/skills/` | Easy |
| [OpenClaw](#openclaw) | YAML | Manual load | Medium |
| [Hermes](#hermes) | `SKILL.md` | `~/.hermes/skills/` | Easy |
| [GitHub Copilot CLI](#github-copilot-cli) | `SKILL.md` | `.agents/skills/` | Easy |
| [Codex CLI](#codex-cli) | `SKILL.md` | `.agents/skills/` | Easy |
| [Cursor](#cursor) | `SKILL.md` + `.mdc` | `.cursor/skills/` | Easy |
| [Windsurf](#windsurf) | `SKILL.md` + `.md` | `.windsurf/skills/` | Easy |
| [Continue](#continue) | `.md` rules | `.continue/rules/` | Medium |
| [Cline](#cline) | `.md` rules | `.clinerules/` | Medium |
| [Aider](#aider) | `CONVENTIONS.md` | Manual `--read` | Medium |

## Cross-Agent Compatibility

Five agents support the **SKILL.md** standard natively:
- Hermes, GitHub Copilot CLI, Codex CLI, Cursor, Windsurf

These agents auto-discover from `.agents/skills/`. The symlink is not tracked:
run `./scripts/setup-agents.sh --agent all` to create it.

## Skill Bundle Contents

The skill bundle (`skills/SKILL.md` + sub-skills) includes:

- **SKILL.md** — Main skill definition with CLI command reference
- **workflow-removal-cycle.md** — Complete removal lifecycle orchestration guide
- **setup-identity.md** — Identity vault setup
- **plan-removal-campaign.md** — Campaign planning
- **send-removal-batch.md** — Sending removal requests
- **triage-broker-replies.md** — Daily inbox triage workflow
- **handle-action-required.md** — Handling verifications and rejections
- **daily-tick.md** — Running the tick engine
- **re-scan-quarterly.md** — Quarterly re-scan workflow

The **workflow-removal-cycle.md** template ties all sub-skills together into a repeatable cycle: plan → execute → wait → poll → classify → respond → tick → re-scan. It includes a decision matrix for when to use each command and error handling guidance.

## Agent-Specific Setup

### Claude Code

**Format**: `SKILL.md`  
**Path**: `.claude/skills/symaira-eraseme/`  
**Status**: ✅ Already configured (symlink exists)

```bash
cd /path/to/symaira-eraseme
./scripts/setup-agents.sh --agent claude
ls -la .claude/skills/
# symaira-eraseme -> ../../skills
```

See [examples/claude-code/](examples/claude-code/) for details.

### OpenClaw

**Format**: YAML  
**Path**: `~/.config/openclaw/skills/symeraseme.yaml`  
**Status**: Manual install required

```bash
# Copy YAML skill definition
cp examples/openclaw/symeraseme.yaml ~/.config/openclaw/skills/
openclaw skill load symeraseme
```

See [examples/openclaw/](examples/openclaw/) for details.

### Hermes

**Format**: `SKILL.md`  
**Path**: `~/.hermes/skills/privacy-tools/symaira-eraseme/`  
**Status**: Manual install required

```bash
hermes skills install https://raw.githubusercontent.com/danieljustus/Symaira-EraseMe/main/skills/SKILL.md
```

Or manually:
```bash
mkdir -p ~/.hermes/skills/privacy-tools/symaira-eraseme
cp skills/SKILL.md ~/.hermes/skills/privacy-tools/symaira-eraseme/
```

See [examples/hermes/](examples/hermes/) for details.

### GitHub Copilot CLI

**Format**: `SKILL.md`  
**Path**: `.agents/skills/` or `~/.copilot/skills/`  
**Status**: ✅ Auto-discovered from `.agents/skills/`

```bash
./scripts/setup-agents.sh --agent codex
ls -la .agents/skills/
# symaira-eraseme -> ../../skills
```

Verify:
```bash
copilot /skills reload
copilot /skills info symaira-eraseme
```

### Codex CLI

**Format**: `SKILL.md`  
**Path**: `.agents/skills/` or `~/.codex/skills/`  
**Status**: ✅ Auto-discovered from `.agents/skills/`

```bash
# Already configured in this repo
codex /skills reload
codex /skills info symaira-eraseme
```

See [examples/codex/](examples/codex/) for optional metadata file.

### Cursor

**Format**: `SKILL.md` (skills) + `.mdc` (rules)  
**Path**: `.cursor/skills/` or `.agents/skills/`  
**Status**: ✅ Auto-discovered from `.agents/skills/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danieljustus/symaira-eraseme](https://github.com/danieljustus/symaira-eraseme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
