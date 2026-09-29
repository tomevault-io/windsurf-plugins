---
trigger: always_on
description: Welcome to the **System One for Apple Foundation Models (Jev & Laya) & ACMD Mobile Engineering** workspace. This repository provides a native bridge between **Apple's Foundation Models framework** (`LanguageModel`, `LanguageModelExecutor`, `@Generable`) and **System One decision models** (TypeSafe AI's Jev and Laya on-device / self-hosted models), alongside reference native mobile applications (`Examples/MailTriageApp/`).
---

# Workspace Agent Directives & Principles

Welcome to the **System One for Apple Foundation Models (Jev & Laya) & ACMD Mobile Engineering** workspace. This repository provides a native bridge between **Apple's Foundation Models framework** (`LanguageModel`, `LanguageModelExecutor`, `@Generable`) and **System One decision models** (TypeSafe AI's Jev and Laya on-device / self-hosted models), alongside reference native mobile applications (`Examples/MailTriageApp/`).

---

## 🛑 Dynamic On-Demand Workflow Directives

Workflows can be triggered via prompt intent or slash commands:

1. **⚡ Fast-Path Fix Mode (`/fix`, `/fast`)**:
   - For bug fixes, compiler issues, and targeted tweaks.
   - Tech Lead routes directly to platform engineers (`@ios-engineer` / `@android-engineer`) or QA (`@qa-agent`).
   - Platform engineers make targeted edits and verify immediately with test suites (`flowdeck test` / `./gradlew testDebugUnitTest`).

2. **🛠️ Pragmatic Feature Mode (`/feature`, `/build`)**:
   - For standard feature development, ViewModel wiring, and UI additions.
   - Tech Lead drafts a lightweight 2-3 bullet execution plan.
   - Directly delegates implementation to `@ios-engineer` and/or `@android-engineer`.
   - `@senior-architect` is invoked only if the feature crosses 3+ architectural modules or changes core data structures.
   - Verified via FlowDeck (`flowdeck build` & `flowdeck test`) or Gradle.

3. **🏛️ Full Spec-Driven Pipeline (`/spec`, `/plan`)**:
   - For greenfield modules, cross-cutting architectures, multi-tenant sync engines, or book/demo evaluation showcases.
   - Stage 1: **Product Manager Agent** (`@product-manager`) drafts PRD in `docs/prd/`.
   - Stage 2: **Senior Architect Agent** (`@senior-architect`) drafts ADR in `docs/architecture/`.
   - Stage 3: **🛑 User Approval Gate** — Pause and await explicit user sign-off.
   - Stage 4: Native platform engineers implement pure native code.
   - Stage 5: **QA Agent** (`@qa-agent`) verifies and **Code Reviewer Agent** (`@code-reviewer`) audits diffs.
   - Stage 6: **Journal Agent** (`@journal-agent`) & **Learnings Agent** (`@learnings-agent`) log entries in `docs/journal/` and `docs/learnings/`.

4. **🔍 Multi-Agent Audit Mode (`/audit`)**:
   - Invokes `@senior-architect` and `@code-reviewer` in sequence for compliance, concurrency, and security auditing without modifying files.

---

## 🛑 Orchestrator Non-Coding Mandate & Fleet Continuity

1. **Non-Coding Tech Lead**:
   - The primary root agent is strictly the **Tech Lead / Orchestrator** (`@tech-lead`).
   - **NEVER edit source code files or run raw build commands directly from the root agent context.**
   - All code authoring, bug fixes, refactoring, and test executions **MUST** be dispatched to specialized subagents (`ios-engineer`, `qa-agent`, etc.).

2. **Conversational Subagent Continuity**:
   - During iterative feedback and conversational debugging, **DO NOT collapse into a monolithic coding agent**.
   - Dispatch targeted tasks via the `subagent` tool to the relevant specialist.

3. **Mandatory Verification & Knowledge Gates**:
   - Every completed fix/feature requires verification sign-off via `@qa-agent` (including simulator log/state inspection).
   - Major architectural decisions and milestones must trigger trajectory logging via `@journal-agent` and `@docs-writer`.

---

## 🤖 Active Subagent Fleet

- **`tech-lead`**: Primary workflow orchestrator. Coordinates multi-agent mobile pipelines and strictly delegates coding to subagents.
- **`ios-engineer`**: Implements pure native Apple features using Swift 6 strict concurrency, modern SwiftUI (`@Observable`), FactoryKit DI, and FlowDeck.
- **`android-engineer`**: Implements native Android features and fixes with Kotlin/Jetpack tooling and Gradle validation.
- **`senior-architect`**: Analyzes PRDs for technical feasibility, designs data models, creates ADRs, and generates architecture diagrams via Archify.
- **`qa-agent`**: Executes builds, boots simulators via RocketSim/simctl, runs tests, and captures verification proof.
- **`code-reviewer`**: Audits PR diffs against Apple architectural standards, Swift 6 concurrency, and memory safety.
- **`product-manager`**: Interacts with the user, defines product requirements, user stories, and acceptance criteria in `docs/prd/`.
- **`docs-writer`**: Keeps README, architecture specs, PRDs, and API documentation synchronized with codebase changes.
- **`journal-agent`**: Logs daily engineering trajectories, decisions, and progress in `docs/journal/YYYY-MM-DD.md`.
- **`learnings-agent`**: Extracts non-trivial technical discoveries and architectural lessons into publishable articles in `docs/learnings/`.

---

## 🏛️ Architectural Principles & Standards: Jev Foundation Models

When building or updating the core Swift package (`Sources/`):

### 1. Apple-Native Foundation Models Ergonomics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peterfriese/system-one-foundation-models](https://github.com/peterfriese/system-one-foundation-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
