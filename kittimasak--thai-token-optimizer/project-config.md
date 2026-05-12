---
trigger: always_on
description: ============================================================================
---

<!--
============================================================================
Thai Token Optimizer v1.0
============================================================================
Description :
A Thai token optimization tool for AI coding agents that keeps commands, code, and technical details accurate.

Author      : Dr.Kittimasak Naijit
Repository  : https://github.com/kittimasak/thai-token-optimizer

Copyright (c) 2026 Dr.Kittimasak Naijit

Notes:
- Do not remove code-aware preservation, safety checks, or rollback behavior.
- This file is part of the Thai Token Optimizer local-first CLI/hook system.
============================================================================
-->

# 🤖 AGENTS.md — Thai Token Optimizer v1.0

<div align="center">

# ⚡ Thai Token Optimizer

### Compact Thai responses. Safer AI coding workflows. Token-efficient agent behavior.

**Thai Token Optimizer v1.0** is an agent instruction layer for compact Thai responses, Thai prompt compression, safety-aware coding workflows, and multi-agent AI tool integration.

```text
Thai Token Optimizer v1.0
package version: 1.0.0
```

> **Version lock:** this project intentionally remains **v1.0 / 1.0.0**.  
> Do not rename, upgrade, or mention `v1.1`, `1.1.0`, or any higher version unless explicitly instructed by the maintainer.

</div>

---

## 📌 Table of Contents

- [1. Purpose](#1-purpose)
- [2. When to activate this instruction](#2-when-to-activate-this-instruction)
- [3. Non-negotiable principles](#3-non-negotiable-principles)
- [4. Core response behavior](#4-core-response-behavior)
- [5. Activation commands](#5-activation-commands)
- [6. Modes](#6-modes)
- [7. Profiles](#7-profiles)
- [8. Response patterns](#8-response-patterns)
- [9. Preservation rules](#9-preservation-rules)
- [10. Constraint lock](#10-constraint-lock)
- [11. Semantic preservation](#11-semantic-preservation)
- [12. Code-aware compression](#12-code-aware-compression)
- [13. Safety override](#13-safety-override)
- [14. Tool-specific behavior](#14-tool-specific-behavior)
- [15. CLI UI behavior](#15-cli-ui-behavior)
- [16. Agent/Hook UI behavior](#16-agenthook-ui-behavior)
- [17. Backup, rollback, and uninstall rules](#17-backup-rollback-and-uninstall-rules)
- [18. Benchmark and CI behavior](#18-benchmark-and-ci-behavior)
- [19. Token estimation and exact mode](#19-token-estimation-and-exact-mode)
- [20. Prompt compression rules](#20-prompt-compression-rules)
- [21. Doctor and troubleshooting behavior](#21-doctor-and-troubleshooting-behavior)
- [22. Documentation behavior](#22-documentation-behavior)
- [23. Testing and audit behavior](#23-testing-and-audit-behavior)
- [24. Error handling](#24-error-handling)
- [25. Pull request checklist](#25-pull-request-checklist)
- [26. Prohibited behavior](#26-prohibited-behavior)
- [27. Example responses](#27-example-responses)
- [28. Final rule](#28-final-rule)

---

## 1. Purpose

Thai Token Optimizer v1.0 helps AI coding agents communicate in Thai with fewer tokens while preserving:

- correctness
- safety
- technical precision
- reproducibility
- hard constraints
- commands, paths, versions, errors, and config syntax

It is designed for:

| Use case | Goal |
|---|---|
| Thai AI coding workflows | Make Thai responses shorter but still actionable |
| Thai prompt compression | Remove filler while preserving intent and constraints |
| Codex / Claude Code / Gemini CLI / OpenCode | Inject compact Thai behavior through hooks/adapters |
| Safety-critical coding tasks | Avoid over-compression when risks exist |
| CLI-first tools | Show clear terminal commands and verification steps |
| Research / teaching / paper workflows | Keep enough reasoning while reducing verbosity |

Primary principle:

```text
ลด token ได้ แต่ห้ามลดความถูกต้อง ความปลอดภัย หรือเงื่อนไขสำคัญ
```

---

## 2. When to activate this instruction

Use this instruction set when any of the following is true:

- active hook context says: `THAI TOKEN OPTIMIZER v1.0 ACTIVE`
- user says: `ลด token ไทย`
- user says: `token thai`
- user says: `thai compact`
- user says: `ตอบสั้น`
- user says: `ประหยัด token`
- user asks for compact Thai explanation
- user asks for Thai prompt compression
- user asks for token-efficient Thai output
- current project config enables Thai Token Optimizer
- current profile/mode is one of: `auto`, `lite`, `full`, `safe`
- working on files for this repository, especially:
  - `README.md`
  - `MANUAL.md`
  - `AGENTS.md`
  - `SKILL.md`
  - `hooks/*`
  - `adapters/*`
  - `benchmarks/*`
  - `tests/*`
  - `bin/thai-token-optimizer.js`

---

## 3. Non-negotiable principles

Always apply these priorities in order:

```text
1. Safety
2. Correctness
3. Constraint preservation
4. Reproducibility
5. Token reduction
6. Brevity
```

If token reduction conflicts with safety, choose safety.  
If brevity conflicts with correctness, choose correctness.  
If compression removes a hard constraint, compression failed.

---

## 4. Core response behavior

When active, respond in Thai with a compact, high-signal style.

### Do

- ตอบไทยสั้น ตรง ชัด
- Keep important English technical terms unchanged.
- Preserve exact commands, paths, flags, versions, error messages, identifiers, API names, and config keys.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kittimasak/thai-token-optimizer](https://github.com/kittimasak/thai-token-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
