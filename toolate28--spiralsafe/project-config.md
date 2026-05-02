---
trigger: always_on
description: Purpose: Short, actionable instructions to help AI coding agents be immediately productive in this repository.
---

# Copilot / Agent Quick Guide for SpiralSafe

Purpose: Short, actionable instructions to help AI coding agents be immediately productive in this repository.

## Start here (big-picture)

- Read: `ARCHITECTURE.md` (high-level layers) and `.github/AGENTS.md` (agent roles & coordination).
- Key design artifacts: `protocol/wave-spec.md`, `protocol/bump-spec.md`, `protocol/sphinx-spec.md`, `methodology/atom.md` (how work is chunked), and `foundation/*` for theory.
- Related repos: `wave-toolkit` (coherence), `kenl` (ATOM trail), `quantum-redstone`, `coherence-mcp`, `spiralsafe-mono`, `QDI`.

## Protocol Stack (WAVE → SPHINX → BUMP → ATOM)

```
WAVE ──► SPHINX ──► BUMP ──► ATOM
         │
         ├─ ORIGIN:     "Where do you come from?"
         ├─ INTENT:     "What do you seek?"
         ├─ COHERENCE:  "Are you whole?" (>60%)
         ├─ IDENTITY:   "Who are you?"
         └─ PASSAGE:    "You may pass."
```

### SPHINX Protocol (Guardian Gates)
The **S**ecure **P**rotocol for **H**ierarchical **I**dentity, **N**avigation, and e**X**change guards all transitions between system boundaries.

| Gate | Marker | Function | Riddle |
|------|--------|----------|--------|
| ORIGIN | `SPHINX:ORIGIN` | Verify genesis lineage | "Where do you come from?" |
| INTENT | `SPHINX:INTENT` | Verify stated purpose | "What do you seek?" |
| COHERENCE | `SPHINX:COHERENCE` | Verify >60% threshold | "Are you whole?" |
| IDENTITY | `SPHINX:IDENTITY` | Verify agent/user auth | "Who are you?" |
| PASSAGE | `SPHINX:PASSAGE` | Grant transition | "You may pass." |

**Syntax:**
```markdown
<!-- SPHINX:COHERENCE
  threshold: 0.6
  source: wave_analyze
  result: 0.72
  verdict: PASSAGE
-->
Coherence verified. Proceeding to BUMP handoff.
<!-- /SPHINX:COHERENCE -->
```

### Self-Referential Loop Termination
- Each SPHINX gate logs its own passage to ATOM trail
- New loops require distinct, auditable genesis events
- No action can cycle without satisfying all gate riddles
- Surjection supported: every output maps to a verifiable input

## What to read first for a task

- For protocol or handoff changes: `protocol/bump-spec.md` and `.context.yaml` examples.
- For documentation coherence work: `protocol/wave-spec.md` and `project-book.ipynb`.
- For ops & verification helpers: `ops/README.md`, `ops/scripts/session_report.py`, `ops/scripts/sign_verification.py`.

## Local dev & CI (how to run things)

- Node (repo uses Node 20 in CI). Common commands:
  - npm ci
  - npm run typecheck
  - npm run lint
  - npm test
  - npm run build
- Lint & static analysis:
  - Shell scripts: `shellcheck` (pre-commit and CI)
  - PowerShell: `PSScriptAnalyzer` (invoked in CI)
- CI specifics: `.github/workflows/spiralsafe-ci.yml` runs coherence (wave) analysis, SPHINX gate checks, then lint/typecheck/tests.

## Project-specific patterns & conventions (must follow)

- **H&&S markers** (protocol/bump-spec.md): use `H&&S:WAVE` for soft handoff (add to PR body for review), `H&&S:PASS` to transfer ownership, `H&&S:SYNC` for synchronization, `H&&S:BLOCK` for blocking issues, `H&&S:GH-COPILOT` for GitHub Copilot agent signatures. Examples: include `H&&S:WAVE` in PR body for architectural changes.
- **Commit message format**: `[layer] Brief description` (layers e.g. `[protocol]`, `[interface]`, `[methodology]`). See `CONTRIBUTING.md`.
- **ATOM tagging**: Format `ATOM-TYPE-YYYYMMDD-NNN-description`. Types: INIT, FEATURE, FIX, DOC, REFACTOR, TEST, DECISION, RELEASE, TASK.
  - ATOM tags in commit messages are automatically extracted and logged to the SpiralSafe API
  - Regex pattern: `ATOM-[A-Z]+-[0-9]{8}-[0-9]{3}-[a-z0-9-]+`
- **PR body markers for API logging**: Use HTML comments to control bump marker creation:
  - `<!-- spiralsafe:bump:TYPE:state -->` - Explicit bump type override (e.g., `<!-- spiralsafe:bump:WAVE:pr_opened -->`)
  - Automatically detected: PR opened → WAVE, PR merged → PASS, PR closed → SYNC, PR updated → PING
- **Dual-format docs**: Many files follow a dual-format convention—prose + structured summary (`.context.yaml` style). Preserve both when adding or editing docs.
- **Atom trail**: project sessions and decisions live in `.atom-trail/` (subdirs: `decisions`, `sessions`, `verifications`). Use `ops/scripts/session_report.py` (`start` / `signout`) for session work, and `ops/scripts/sign_verification.py` to record human signatures.
- **Verification & signing**:
  - Start a session: `python ops/scripts/session_report.py start "desc"`
  - Sign out: `python ops/scripts/session_report.py signout <ATOM_TAG>`
  - Add verification: `python ops/scripts/sign_verification.py <VER_TAG> --name "your-name"`
  - Encryption helper: `ops/scripts/Transcript-Pipeline.ps1` (PowerShell, AES-256-GCM)
  - Verify signatures: documented as `ss-verify <path>` in docs (see `ops/DEPLOYMENT_ARCHITECTURE.md`).

## Code style guidelines

### Shell Scripts

- Always use `set -euo pipefail` for strict mode
- Check dependencies gracefully before use
- Provide clear error messages with recovery steps
- Make scripts idempotent when possible

### PowerShell Scripts

- Use `#Requires -Version 5.1` at the top
- Set `$ErrorActionPreference = "Stop"`
- Use appropriate Write-\* cmdlets (Write-Host for user output, Write-Error for errors)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toolate28/SpiralSafe](https://github.com/toolate28/SpiralSafe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
