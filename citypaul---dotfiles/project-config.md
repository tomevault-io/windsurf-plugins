---
trigger: always_on
description: > **About this file (v3.0.0):** Lean version optimized for context efficiency. Core principles here; detailed patterns loaded on-demand via skills.
---

# Development Guidelines for Claude

> **About this file (v3.0.0):** Lean version optimized for context efficiency. Core principles here; detailed patterns loaded on-demand via skills.
>
> **Architecture:**
> - **CLAUDE.md** (this file): Core philosophy + quick reference (~160 lines, always loaded)
> - **Skills**: Detailed patterns loaded on-demand (specification, ubiquitous-language, tdd, testing, mutation-testing, test-design-reviewer, typescript-strict, functional, refactoring, reduce-system-complexity, expectations, planning, story-splitting, front-end-testing, react-testing, ci-debugging, hexagonal-architecture, domain-driven-design, event-sourcing, twelve-factor, api-design, bff-design, bff-entry-points, cli-design, codebase-design, improve-codebase-architecture, structure-codebase, evaluate-existing-solutions, finding-seams, characterisation-tests, production-parity-skill-builder, storyboard, teach-me, diagrams, technical-writing, find-skills, find-gaps, double-check)
> - **External skills**: Loaded on-demand from community repos (impeccable + 17 steering commands from [pbakaus/impeccable](https://github.com/pbakaus/impeccable), 6 web quality skills from [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills), 3 Next.js skills from [vercel-labs/next-skills](https://skills.sh/vercel-labs/next-skills), grill-me from [mattpocock/skills](https://skills.sh/mattpocock/skills/grill-me), seo-audit from [coreyhaines31/marketingskills](https://skills.sh/coreyhaines31/marketingskills/seo-audit))
> - **Agents**: Specialized subprocesses for verification and analysis
>
> **Previous versions:**
> - v2.0.0: Modular with @docs/ imports (~3000+ lines always loaded)
> - v1.0.0: Single monolithic file (1,818 lines)

## Core Philosophy

**TEST-DRIVEN DEVELOPMENT IS NON-NEGOTIABLE FOR NEW OR CHANGED BEHAVIOR.** Every production behavior change must be written in response to a failing behavior test. Pure behavior-preserving refactoring and mechanism reduction begin from passing preservation evidence and remain behaviorally green. Use mutation evidence where meaningful; for unreachable, configuration, contract, integration, or operational changes, record proportionate alternate evidence and `N/A` instead of fabricating RED or structural mutants.

I follow Test-Driven Development (TDD) with a strong emphasis on behavior-driven testing and functional programming principles. All work should be done in small, incremental changes that maintain a working state throughout development.

## Quick Reference

**Key Principles:**

- Write behavior tests first for new or changed behavior (TDD)
- Test behavior, not implementation
- No `any` types or type assertions
- Immutable data only
- Small, pure functions
- TypeScript strict mode always
- Use real schemas/types in tests, never redefine them

**Preferred Tools:**

- **Language**: TypeScript (strict mode)
- **Testing**: Vitest (prefer Browser Mode for UI tests) + Testing Library
- **State Management**: Prefer immutable patterns

## Testing Principles

**Core principle**: Test behavior, not implementation. 100% coverage through business behavior.

**Quick reference:**
- Write behavior tests first for new or changed behavior (TDD non-negotiable)
- Test through the subject's public interface at the layer the claim names (an HTTP endpoint is the wrong interface for a browser claim)
- Use factory functions for test data (no `let`/`beforeEach`)
- Tests must document expected business behavior
- No 1:1 mapping between test files and implementation files

For detailed testing patterns and examples, load the `testing` skill.
For verifying test effectiveness through mutation analysis, load the `mutation-testing` skill.

## TypeScript Guidelines

**Core principle**: Strict mode always. Schema-first at trust boundaries, types for internal logic.

**Quick reference:**
- No `any` types - ever (use `unknown` if type truly unknown)
- No type assertions without justification
- Prefer `type` over `interface` for data structures
- Reserve `interface` for behavior contracts only
- Define schemas first, derive types from them (Zod/Standard Schema)
- Use schemas at trust boundaries, plain types for internal logic

For detailed TypeScript patterns and rationale, load the `typescript-strict` skill.
For API and interface design patterns, load the `api-design` skill.
For OAuth 2.0 or OpenID Connect design, implementation, review, testing, incident analysis, or migration, load the `secure-oauth-oidc` skill.

## Code Style

**Core principle**: Functional programming with immutable data. Self-documenting code.

**Quick reference:**
- No data mutation - immutable data structures only
- Pure functions wherever possible
- No nested if/else - use early returns or composition
- No comments - code should be self-documenting
- Prefer options objects over positional parameters
- Use array methods (`map`, `filter`, `reduce`) over loops
- Compose small private functions behind cohesive, stable module contracts; do not equate one helper with one public module

For detailed patterns and examples, load the `functional` skill.

## Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [citypaul/.dotfiles](https://github.com/citypaul/.dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
