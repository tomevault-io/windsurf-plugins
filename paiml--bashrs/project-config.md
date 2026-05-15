---
trigger: always_on
description: Transforms non-deterministic bash ($RANDOM, timestamps) into safe, idempotent POSIX sh with -p/-f flags.
---

# CLAUDE.md - Rash Development Guidelines

## CRITICAL: Contract-First Design

**NEVER write code before writing a provable contract.**

All code changes MUST have a corresponding contract (YAML in ../provable-contracts/contracts/<project>/ or .pmat-work/<TICKET>/contract.json) BEFORE implementation. This is enforced by `pmat comply` CB-1400.

- Use `pmat comply check` to verify contract coverage
- Minimum verification level: L1 (recommended L3+)
- See docs/agent-instructions/provable-contract-first-agents.md for the full workflow

## Project Context
**Rash (bashrs)** is a shell safety and purification tool:

### 🚨 IMPLEMENTATION STATUS (Updated: 2024-10-18)

**What's IMPLEMENTED (v1.4.0)**:
- ✅ **Bash → Purified Bash**: Parse bash scripts and output deterministic, idempotent POSIX sh (70% production-ready)
- ✅ **Makefile Purification**: Parse and purify Makefiles (v1.4.0)
- ✅ **Security Linter**: 8 critical security rules (SEC001-SEC008)
- ✅ **Determinism/Idempotency Rules**: 6 DET/IDEM linter rules

**What's PLANNED (v3.0+)**:
- ⏸️ **Rust → Safe Shell**: Write Rust code, transpile to shell (infrastructure partial, not production-ready)
- ⏸️ **Full Linter**: 800+ rules (current: 14 rules, 1.75% complete)

---

### PRIMARY WORKFLOW: Bash → Purified Bash (v1.4.0 - WORKING)

Ingest messy bash scripts and output purified, safe, deterministic POSIX shell.

**Purification pipeline**:
1. Parse legacy bash (with $RANDOM, timestamps, non-idempotent code)
2. Apply semantic transformations (determinism + idempotency enforcement)
3. Generate purified POSIX sh (safe to re-run, deterministic)

**Why this is valuable**:
- Clean up existing bash scripts automatically
- Remove non-deterministic patterns ($RANDOM, timestamps, process IDs)
- Make operations idempotent (mkdir -p, rm -f, ln -sf)
- Ensure POSIX compliance (passes shellcheck)
- Quote all variables for injection safety

**Current Status**: 70% production-ready (needs docs, examples, performance tuning)

---

### FUTURE WORKFLOW: Rust → Safe Shell (v3.0+ - PLANNED)

**Vision**: Write REAL Rust code, test with standard Rust tooling, then transpile to provably safe, deterministic POSIX shell scripts.

**Current State**:
- ⚠️ Partial stdlib infrastructure (function name mappings exist)
- ❌ Rust parser/analyzer not implemented
- ❌ Rust std → shell implementation incomplete
- ❌ No production examples or tests

**Estimated Work**: 12-16 weeks from current state

This workflow is **deferred to v3.0+** to focus on completing the working Bash purifier.

---

## Workflow 1: Bash → Purified Bash (PRIMARY - WORKING)

Transforms non-deterministic bash ($RANDOM, timestamps) into safe, idempotent POSIX sh with -p/-f flags.

---


---

## Code Search (pmat query)

**NEVER use grep or rg for code discovery.** Use `pmat query` instead -- it returns quality-annotated, ranked results with TDG scores and fault annotations.

```bash
# Find functions by intent
pmat query "shell ast parsing" --limit 10

# Find high-quality code
pmat query "bash builtin" --min-grade A --exclude-tests

# Find with fault annotations (unwrap, panic, unsafe, etc.)
pmat query "command execution" --faults

# Filter by complexity
pmat query "pipe handling" --max-complexity 10

# Cross-project search
pmat query "rust codegen" --include-project ../depyler

# Git history search (find code by commit intent via RRF fusion)
pmat query "fix redirect handling" -G
pmat query "fix redirect handling" --git-history

# Enrichment flags (combine freely)
pmat query "parser" --churn              # git volatility (commit count, churn score)
pmat query "builtin" --duplicates          # code clone detection (MinHash+LSH)
pmat query "command handler" --entropy             # pattern diversity (repetitive vs unique)
pmat query "shell transpilation" --churn --duplicates --entropy --faults -G  # full audit
```

## Development Principles

### EXTREME TDD Definition

**EXTREME TDD** is traditional Test-Driven Development enhanced with comprehensive quality gates:

**Formula**: EXTREME TDD = TDD + Property Testing + Mutation Testing + Fuzz Testing + PMAT + Examples

**Components**:
1. **TDD (RED → GREEN → REFACTOR)**: Write failing test → Implement → Clean up
2. **Property Testing**: Generative tests with 100+ cases (proptest)
3. **Mutation Testing**: Verify test quality with ≥90% kill rate (cargo-mutants)
4. **Fuzz Testing**: Automated input generation to find edge cases (when applicable)
5. **PMAT Quality Gates**: Code complexity (<10), quality score (≥9.0), TDG verification
6. **Example Verification**: `cargo run --example` must pass for all relevant examples


### Quality Targets
- Test coverage >95%, complexity <10
- Purified scripts pass shellcheck
- Performance: <100ms transpilation, <10MB memory

---

## 🚨 STOP THE LINE Protocol (Andon Cord)

**CRITICAL**: When validating GNU Bash Manual transformations (Workflow 2), **STOP THE LINE** immediately when a bug is discovered.

### When to Pull the Andon Cord

**STOP IMMEDIATELY** if you discover:
1. ❌ **Missing implementation** - Bash construct not parsed correctly
2. ❌ **Incorrect transformation** - Bash→Rust or Rust→Purified output is wrong
3. ❌ **Non-deterministic output** - Purified bash contains $RANDOM, $$, timestamps, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paiml/bashrs](https://github.com/paiml/bashrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
