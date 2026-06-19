---
trigger: always_on
description: |
---


<!-- Threat Modeling Skill | Version 3.2.0 (20260512a) | https://github.com/fr33d3m0n/threat-modeling | License: BSD-3-Clause -->

> **Note**: All relative paths in this skill are relative to `SKILL_PATH` / `CLAUDE_SKILL_DIR` (the directory containing this SKILL.md file). Use `${SKILL_PATH:-$CLAUDE_SKILL_DIR}` in Bash commands for cross-runtime compatibility.

# Threat Modeling Skill v3.2.0 (20260512a)

AI-native automated software risk analysis skill. LLM-driven, Code-First approach for comprehensive security risk assessment, threat modeling, security testing, penetration testing, and compliance checking.

## Version Banner

```
━━━ 🛡️ Threat Modeling Skill v3.2.0 (20260512a) ━━━
```

## Command Line Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--debug` | Publish internal YAML data files, KB queries, coverage validation, and evaluation report | OFF |
| `--lang=xx` | Set output language (en, zh, ja, ko, es, fr, de, pt, ru) | Auto-detect |
| `--detailed` | Auto-trigger P8R (detailed per-VR analysis reports) after P8 completes | OFF |

**Usage Examples**:
```bash
# Default mode - 11 deliverable files only
/threat-model @my-project

# Debug mode - all internal files published
/threat-model @my-project --debug

# Chinese output with debug
/threat-model @my-project --lang=zh --debug

# Generate detailed per-VR analysis reports after P8
/threat-model @my-project --detailed

# Full options
/threat-model @my-project --detailed --debug --lang=zh
```

---

## ⚠️ CRITICAL: Data vs Report Separation

> **PRINCIPLE**: Markdown is for reports (human-readable), YAML is for data (machine-readable). They MUST be separated!

```
┌─────────────────────────────────────────────────────────────────────┐
│  DUAL OUTPUT MODEL - Each phase produces TWO files:                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. DATA FILE (.yaml) - PRIMARY                                     │
│     • Written FIRST                                                  │
│     • Structured, machine-readable                                   │
│     • Used by NEXT phase as input                                    │
│     • Path: .phase_working/{SESSION_ID}/data/P{N}_*.yaml            │
│                                                                      │
│  2. REPORT FILE (.md) - SECONDARY                                   │
│     • Written AFTER data file                                        │
│     • Human-readable, formatted                                      │
│     • For review and documentation                                   │
│     • Path: .phase_working/{SESSION_ID}/reports/P{N}-*.md           │
│                                                                      │
│  ❌ FORBIDDEN: Reading .md files for data extraction                │
│  ❌ FORBIDDEN: Embedding data as yaml blocks inside .md AS SOURCE   │
│  ✅ ALLOWED: YAML blocks in .md for schema documentation/examples   │
│  ✅ REQUIRED: Data flows via .yaml files only                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## §1 Execution Model

**Mode**: Full Assessment Only - All 8 phases executed sequentially.

```
Phase 1 ──► Phase 2 ──► Phase 3 ──► Phase 4 ──► Phase 5 ──► Phase 6 ──► Phase 7 ──► Phase 8
   │            │            │            │            │            │            │
   ▼            ▼            ▼            ▼            ▼            ▼            ▼
P1.yaml ──► P2.yaml ──► P3.yaml ──► P4.yaml ──► P5.yaml ──► P6.yaml ──► P7.yaml ──► P8.yaml
```

**Rules**:
1. Phases execute strictly in order (1→8)
2. Each phase reads previous phase's YAML, writes its own YAML
3. Each phase also writes a human-readable .md report
4. Validation runs on YAML files, not .md files
5. Phase 6 = Risk Validation (NOT mitigation)
6. Phase 7 = Mitigation Planning (AFTER validation)

**Phase Gate Protocol**:
```
FOR each phase N in [1..8]:
    1. Read: @phases/P{N}-*.md (instructions)
    2. Read: .phase_working/{SESSION_ID}/data/P{N-1}_*.yaml (input, except P1)
    3. Execute analysis per phase instructions
    4. Write: .phase_working/{SESSION_ID}/data/P{N}_*.yaml (PRIMARY output)
    5. Write: .phase_working/{SESSION_ID}/reports/P{N}-*.md (SECONDARY output)
    6. Validate: python ${SKILL_PATH:-$CLAUDE_SKILL_DIR}/scripts/phase_data.py --phase-end --phase N --root .
    7. IF exit != 0: Fix YAML and rewrite, re-run --phase-end
    8. IF exit == 0: Update session meta, continue to N+1
```

---

## §2 Output Convention

### Output Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│  OUTPUT MODES - Control what files are generated                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DEFAULT MODE (Production)                                          │
│  ─────────────────────────────────────────────────────────────────  │
│  Only user-deliverable files are published:                         │
│  ✅ 4 Required Reports (RISK-ASSESSMENT, INVENTORY, MITIGATION,    │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fr33d3m0n/threat-modeling](https://github.com/fr33d3m0n/threat-modeling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
