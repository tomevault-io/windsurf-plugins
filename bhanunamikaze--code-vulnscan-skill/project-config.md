---
trigger: always_on
description: Use this when the user wants to find security vulnerabilities in a codebase, perform a security audit, scan for CVEs, detect secrets, review React/Next.js or Go services, audit architecture/application/infrastructure flaws, review auth/API/crypto/business logic, check IaC/cloud/runtime configs, or generate a vulnerability report. Performs taint-flow, control-flow, architecture, and exploitability analysis across Python, JS/TS, Java, Go, PHP, Ruby, C/C++, C#, Rust, and infrastructure.
---


# Code-VulnScan — Deep Codebase Vulnerability Scanner

This skill performs comprehensive, flow-aware security analysis on any codebase. It combines taint tracking, control-flow analysis, business logic review, API security auditing, secret detection, configuration review, and dependency auditing to find real, exploitable vulnerabilities — not keyword matches.

- Use the IDE's own tools for reading, searching, and reasoning about code.
- Use local Python scripts for deterministic file enumeration, AST-based analysis, secret entropy scanning, dependency checking, state tracking, and report generation.
- Do not call external LLM-provider APIs as part of this skill.
- Every confirmed finding requires a verified evidence chain. Candidates without verification are never reported.

## Command surface

- `vulnscan scan <path> [--lang python,javascript,...] [--severity critical,high,medium,low] [--exclude vendor,tests]`
- `vulnscan taint <file> [--lang <language>]`
- `vulnscan secrets <path>`
- `vulnscan deps <path>`
- `vulnscan config <path>`
- `vulnscan report [--run-id <id>] [--format markdown|json|sarif] [--min-severity medium]`
- `vulnscan status`
- `vulnscan commit <hash> [--repo <path>] [--base <base-hash>] [--severity critical,high,medium,low]`
- `vulnscan diff <base> <head> [--repo <path>] [--severity critical,high,medium,low]`
- `vulnscan pr <pr-number> [--repo <path>] [--severity critical,high,medium,low]`

**Commit/diff mode:** Scans only the files and code regions changed in a commit or between two refs. Faster than a full scan — designed for CI/CD pipelines and code review. Findings are tagged `introduced_in_diff: true` (new vulnerability in the changed code) or `introduced_in_diff: false` (pre-existing vulnerability in code called by the change). Use `vulnscan commit HEAD` to scan the most recent commit, `vulnscan diff main HEAD` to scan a feature branch, or `vulnscan pr 42` to scan a GitHub PR diff.

Defaults: report `critical`, `high`, `medium` findings. Auto-detect language from file extensions.

## Core architecture

- `sub-skills/` — cognitive instructions for each analysis phase (24 specialized agents)
- `scripts/` — deterministic Python helpers for enumeration, AST analysis, entropy scanning, dependency checking, report generation
- `resources/patterns/` — per-language source/sink/sanitizer pattern definitions
- `resources/references/` — CWE taxonomy, OWASP Top 10, false-positive guidance
- `workspace/` — SQLite scan state, intermediate JSON outputs, final reports

## The golden rule: evidence-based findings only

A confirmed finding requires **all three**:
1. A **source** — user-controlled data enters the system (or a dangerous condition exists).
2. A **sink / consequence** — a dangerous operation can be triggered.
3. A **path** — source reaches sink with no effective mitigation in between.

Pattern-match candidates are **never** confirmed findings. Every candidate passes through `sub-skills/false-positive-filter.md` before being reported.

## Vulnerability categories covered

| Category | Technique | CWE |
|----------|-----------|-----|
| SQL Injection | Taint + AST | CWE-89 |
| Command Injection | Taint + AST | CWE-78 |
| Path Traversal | Taint + canonicalization check | CWE-22 |
| XSS (Reflected/Stored/DOM) | Taint + output context | CWE-79 |
| SSRF | Taint + URL validation check | CWE-918 |
| Insecure Deserialization | Taint + API check | CWE-502 |
| Server-Side Template Injection | Taint + template API check | CWE-94 |
| Open Redirect | Taint + redirect target check | CWE-601 |
| XXE | Config + parser API check | CWE-611 |
| Auth Bypass / Broken Access Control | Control flow + logic analysis | CWE-287, CWE-285 |
| Broken Authentication | Session + token analysis | CWE-306, CWE-384 |
| IDOR / BOLA | Authorization logic analysis | CWE-639 |
| Mass Assignment | API + model analysis | CWE-915 |
| Business Logic Flaws | Control flow + state analysis | CWE-840 |
| Race Conditions / TOCTOU | Concurrency + file op analysis | CWE-362, CWE-367 |
| Weak Cryptography | Algorithm + key analysis | CWE-327, CWE-326 |
| Hardcoded Secrets | Entropy + pattern detection | CWE-798 |
| Insecure Randomness | RNG API analysis | CWE-338 |
| Dependency CVEs | Manifest + version analysis | CWE-1035 |
| Information Disclosure | Error handling + logging analysis | CWE-209 |
| Security Misconfiguration | Config + header analysis | CWE-16 |
| IaC Misconfigurations | Dockerfile/K8s/Terraform analysis | CWE-732, CWE-284 |
| Memory Safety (C/C++) | Buffer + pointer analysis | CWE-120, CWE-416 |
| ReDoS | Regex complexity analysis | CWE-1333 |
| GraphQL Security | Query depth + introspection check | CWE-284 |
| React / Client-Side App Security | DOM sink + router + storage analysis | CWE-79, CWE-601, CWE-922 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bhanunamikaze/Code-VulnScan-Skill](https://github.com/Bhanunamikaze/Code-VulnScan-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
