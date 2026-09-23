---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Knowledge Compiler** (package: `open-knowledge-compiler`, Apache-2.0) — an open-source system that continuously compiles software engineering artifacts (Git repos, PRs, Jira tickets, docs, OpenAPI specs) into a structured, persistent knowledge base queryable by AI agents. The compiler metaphor is intentional: raw artifacts go in, structured engineering knowledge comes out. The emitted wiki is a conformant [OKF](docs/okf-conformance.md) (Open Knowledge Format) bundle — see [ADR-013](docs/decisions/ADR-013-open-source-okf-conformance.md) for spec-version tracking and the open-source release rationale.

**North star (the moat):** every roadmap and prioritization call should serve one question — *given what this software does and what changed, what exactly must be verified, and how do we know the resulting test is actually trustworthy?* Two capabilities answer it and are this project's actual differentiation from generic repository-intelligence tools (Sourcegraph, JetBrains Context, generic wiki/knowledge compilers like `llm-wiki-compiler`, code-graph tools like `agentforge-graph`): a **Behavioral Contract** model (state/transition/failure-mode knowledge, not just structural knowledge — component/API/dependency facts) for "what must be verified," and a **Test Trust Score** (mutation-kill rate + flakiness + escaped-defect history + coverage completeness, combined into one signal — building on the existing mutation-kill signal, [ADR-018](docs/decisions/ADR-018-stale-test-detection.md) stale-test detection, and the backlogged [ADR-019](docs/decisions/ADR-019-test-flakiness-signal.md)/[ADR-020](docs/decisions/ADR-020-escaped-defect-trust-score.md)) for "is the test trustworthy." Neither exists today as a compiled entity or a synthesized score. When two pieces of work compete for priority, prefer the one that moves toward these over general repository-understanding features — that territory is increasingly commoditized elsewhere.

**Architecture v1.0 is FROZEN (2026-07-18).** The spec set: [docs/vision.md](docs/vision.md), [docs/architecture.md](docs/architecture.md), ADR-001…ADR-010 ([docs/decisions/index.md](docs/decisions/index.md)), [docs/ir.md](docs/ir.md), [docs/data-model.md](docs/data-model.md), [docs/pipeline.md](docs/pipeline.md), [docs/normalize.md](docs/normalize.md). ADRs are immutable — changing a decision requires a superseding ADR; living specs accept additive clarifications only (implementation findings are folded in as marked clarifications). Do not create new architecture documents unless implementation reveals a genuine gap. New (non-superseding) architectural decisions discovered post-freeze are still recorded as new ADRs — see [ADR-011](docs/decisions/ADR-011-cross-repo-dependency-resolution.md) (cross-repo dependency resolution, 2026-07-20), [ADR-012](docs/decisions/ADR-012-defer-verification-requirement-entity.md) (defer VerificationRequirement entity, 2026-07-29), [ADR-013](docs/decisions/ADR-013-open-source-okf-conformance.md) (open-source release + OKF spec-version conformance/migration, 2026-08-05, Proposed). [INITIAL-Brainstorm.md](INITIAL-Brainstorm.md) is the superseded exploratory draft.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kushal-omnius/open-knowledge-compiler](https://github.com/kushal-omnius/open-knowledge-compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
