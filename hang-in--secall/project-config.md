---
trigger: always_on
description: This skill provides a structured workflow for guiding users through collaborative document creation. Act as an active guide, walking users through three stages: Context Gathering, Refinement & Structure, and Reader Testing.
---

# seCall — Agent Instructions

> This file defines project-level rules for all agents in tunaFlow.
> All agents (Claude, Gemini, Codex, OpenCode) must follow these rules.

---

## 1. Project Overview

- Name: seCall
- Status: active
- Language: Rust
- Test: `cargo test`
- Type check: `cargo check`
- Stack: Rust

> Auto-detected by tunaFlow. Verify and adjust if needed.

---

## 2. File Storage Rules

**All documents and artifacts must be created within this project directory.**

- Do NOT create files in `~/.claude/`, `~/.gemini/`, or any external path
- Plans: `docs/plans/`
- Reference docs: `docs/reference/`
- Prompts: `docs/prompts/`
- Code: follow project structure

---

## 3. Documentation Rules

### File Naming
- Short, 2-4 core tokens (camelCase)
- Reference: stable names without dates (e.g., `implementationStatus.md`)
- Plan: `featureNamePlan.md` or `featureNamePlan_YYYY-MM-DD.md`
- Prompt: `docs/prompts/YYYY-MM-DD/short_name.md`

### Document Metadata
- Top of every document: `type`, `status`, `updated_at`
- Status values: `draft` → `in_progress` → `done` → `archived`
- Reference docs: update same file (no date-based duplication)
- Plans/prompts: new documents per task allowed (must update index.md)

### Versioning
- Use `status: archived` + `superseded_by` instead of deletion
- Brainstorm/comparison docs: mark `canonical: false`

---

## 4. Coding Rules

### Language
- Respond in the language the user uses (match user's message language)
- Code, paths, identifiers: keep in original language

### Code Quality
- Only modify what was requested. Do not clean up surrounding code
- Error handling: minimize silent fallbacks during development
- No speculative abstractions or future-proofing
- Modify one path at a time → verify → proceed to next

### Testing
- Verify existing tests pass after changes
- Consider unit tests for new logic

---

## 5. Work Safety Rules

- **Verify replacement works** before removing existing functionality
- **Confirm before destructive operations** (file deletion, schema changes)
- **Single-path modification** — never change multiple execution paths simultaneously
- Check all consumers before modifying shared state

---

## 6. Agent Behavior Rules

- **Plan before implementing** — present your plan and wait for user approval before writing code
- Introduce yourself by profile name first, then engine. No mixed expressions
- Do not claim ownership of other agents' messages
- Respond in the user's language
- Lead with conclusions, then reasoning

---

## 7. Current Status

### Completed
- P18 Rev.2 — 세션 분류 regex 사전 컴파일 및 에러 전파
- P22 Rev.2 — Wiki 파이프라인 (Haiku 생성, lint, review + auto-retry)
- Semantic graph extraction — 694세션 완료 (348 skipped, 0 failed)
- P23 — Store/Search 모듈 경계 리팩토링 (search 모듈에서 SQL 분리)
- P24 — GitHub 이슈 일괄 수정 (#19 timezone, #21 local-only, #22 compact turn, #23 FTS5 중복)
- PR #20 — OpenVINO embedding backend (외부 기여, CoLuthien)
- P25 Phase 0-1 — REST API 서버 + Obsidian 플러그인 MVP (PR #24)
- P25 Phase 2 — 데일리 노트 자동 생성 + Graph 탐색 뷰 (PR #27)
- P27 — BM25-only 선택 시 graph semantic 자동 비활성화 (#25 fix, PR #27)

### In Progress
- P26 — Gemini API 백엔드 추가 (시맨틱 그래프 + Log 일기) — 플랜 문서 작성됨

### Known Issues
- 기존 DB에 FTS 중복 잔존 (--force reingest로 세션별 정리 가능)
- Issue #26 — Codex wiki 백엔드 추가 (외부 기여 PR 요청 중)

---

## 8. Next Priorities

1. (P1) P26 — Gemini API 백엔드 추가 (wiki 작성은 Gemini Pro 3/3.1 예정)
2. (P2) Wiki 파이프라인 대규모 실행 검증 — Gemini 백엔드 완성 후 전체 세션 처리
3. (P2) 테스트 갭 대응 — REST API DTO/라우터 등 미테스트 46건

---

## 9. Git Workflow

- 새 기능/수정은 **feature branch**에서 작업 → PR → merge 패턴을 따름
- 브랜치명: `feat/p25-wiki-dryrun`, `fix/issue-name` 등
- PR에서 관련 이슈 자동 close: `Fixes #19, #21` 등 사용
- 외부 기여 PR은 리뷰 후 필요 시 직접 수정 커밋 추가 → merge → 코멘트

<!-- tunaflow:context-start -->
Project: /Users/d9ng/privateProject/seCall

You are an agent in tunaFlow, a multi-agent orchestration platform.

## tunaFlow Workflow Rules
- When proposing a plan, use <!-- tunaflow:plan-proposal --> markers in your response.
- **Do NOT write files to docs/plans/ until AFTER the plan is promoted by the user.** The promotion happens when the user clicks the promote button on PlanProposalCard.
- After promotion, write plan documents directly in docs/plans/:
- {slug}.md — main plan document
- {slug}-task-NN.md — per-subtask work instruction
- Your role-specific instructions are in docs/agents/{role}.md. Follow them.
- The current plan document (if any) is provided in the context below.
- **If a plan already exists for this conversation, do NOT create a new one.** Instead, propose revisions to the existing plan.

## Architect Rules
- Before writing subtasks, explore the codebase using available tools (rawq search, code-review-graph) to identify exact files and functions.
- Each subtask work instruction (task-NN.md) MUST include these 5 sections:
1. **Changed files** — exact paths verified against the codebase (e.g. src/api/chat.post.ts:42). New files: state explicitly.
2. **Change description** — what to add/modify/remove and why
3. **Dependencies** — which tasks must complete first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hang-in/seCall](https://github.com/hang-in/seCall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
