---
trigger: always_on
description: > This file consolidates all rules from the minimal configuration. Gemini CLI auto-loads this as persistent context. All rules are always active — language-specific rules auto-apply based on file extensions in scope.
---

# Project Engineering Standards

> This file consolidates all rules from the minimal configuration. Gemini CLI auto-loads this as persistent context. All rules are always active — language-specific rules auto-apply based on file extensions in scope.

---

## Rule Priority (Conflict Resolution)

### Priority Order (highest to lowest)
1. **Security Mandate** — always wins
2. **Rugged Software Constitution** — foundational defensibility
3. **Code Completion Mandate** + **Logging Mandate** — validation and instrumentation non-negotiable
4. **Testability-First Design** — maintainability enables future
5. **Feature-specific** — language idioms, concurrency, CI/CD. Higher-priority rules win on conflict.
6. **PRD-gated** — feature-flags, gitops-kubernetes. Only when PRD explicitly requires. Confirm before activating.
7. **YAGNI / KISS** — only when no security/reliability/maintainability trade-off

### Common Conflicts

| Conflict | Resolution |
|---|---|
| YAGNI vs Security | Security wins. Input validation always needed. |
| KISS vs Testability | Testability wins. Interfaces enable testing. |
| Perf vs YAGNI | Measure first. Optimize only after profiling. |
| DRY vs Clarity | Clarity wins until 3+ duplications (Rule of Three). |
| Speed vs Logging | Logging wins. Silent failures = enemy. |
| YAGNI vs PRD-gated | PRD wins if explicitly required. |

When in doubt: *"Which choice = more defensible and maintainable?"* If equal -> simpler one (KISS).

---

## Security Mandate

Security = foundational requirement, not feature.

### Universal Principles
1. **Never trust user input** — validate all data from users/APIs/external sources server-side
2. **Deny by default** — explicit permission grants, never assume access
3. **Fail securely** — fail closed (deny) on errors, never open
4. **Defense in depth** — multiple layers, never single control

For implementation details (auth, validation, queries): see Security Principles section below.

---

## Rugged Software Constitution

### Core Philosophy
"I recognize that my code will be attacked." Generate defensibility, not just functionality.

### Commitments
1. **Responsible** — no happy-path-only code. Every input assumed malformed/malicious. Error handling = first-class feature.
2. **Defensible** — validate own state/inputs (Paranoid Programming). Fail securely (closed). Verify assumptions explicitly.
3. **Maintainable** — write for next year's reader, not today's compiler. Clarity over cleverness. Isolate complexity.

### 7 Rugged Habits
1. **Defense-in-depth** — validate at every boundary (API, DB, fn call). Never single-layer protection.
2. **Instrument for awareness** — code signals attacks/failures. Silent failures = enemy #1.
3. **Reduce attack surface** — remove unused code/deps/endpoints. Minimum public interface (Least Privilege).
4. **Design for failure** — assume DB down, network timeout, disk full. Circuit breakers, fallbacks.
5. **Clean up** — own acquired resources, ensure release. No TODO for security holes; fix or document risk.
6. **Verify defenses** — test unhappy paths as rigorously as happy.
7. **Adapt to ecosystem** — battle-tested libraries over custom. Community conventions for maintainability.

### Code Generation Rules
- **Refuse** insecure patterns (SQLi, hardcoded secrets, shell injection) even if asked.
- **Proactively** add validation, error handling, timeout logic even if not requested.
- **Explain** why defensive measures added.

---

## Code Completion Mandate

**Before marking any code task complete, run automated quality checks and remediate all issues.**

### Completion Workflow
1. **Generate** — write code
2. **Validate** — run language-appropriate quality checks
3. **Remediate** — fix all issues
4. **Verify** — re-run checks
5. **Deliver** — mark complete only after all checks pass

Never skip validation "to save time." Validation IS the work.

### Quality Commands per Language

| Language | Section |
|---|---|
| Go | `go vet ./...`, `staticcheck ./...`, `go test ./...` |
| TypeScript / Vue | `npx tsc --noEmit`, `npx eslint .`, `npm test` |
| Flutter / Dart | `dart analyze`, `flutter test` |
| Rust | `cargo clippy -- -D warnings`, `cargo test` |
| Python | `ruff check .`, `mypy .`, `pytest` |

### Failure Protocol
1. Read error output completely
2. Fix identified issues
3. Re-run failing command
4. Do not proceed until all checks pass

Never disable a lint rule or suppress a warning to pass. Fix root cause.

---

## Core Design Principles

### SOLID
- **SRP** — one reason to change per class/module/fn. If description needs "and" -> violates SRP.
- **OCP** — open for extension, closed for modification. Use composition + DI.
- **LSP** — subtypes substitutable for base types without breaking correctness.
- **ISP** — many small focused interfaces over one monolithic.
- **DIP** — depend on abstractions, not concretions. Core principle for testability-first.

### Essential Practices
- **DRY** — single authoritative representation. No duplicate logic/algorithms/business rules.
- **YAGNI** — no speculative features. Build for today, refactor when needs change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irahardianto/pathfinder](https://github.com/irahardianto/pathfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
