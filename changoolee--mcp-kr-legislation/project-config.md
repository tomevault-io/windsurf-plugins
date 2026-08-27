---
trigger: always_on
description: > **Purpose**: Complete guide for Cursor AI to work effectively on MCP Korean Legislation
---

# MCP 한국 법령 — AI Coding Guidelines

> **Purpose**: Complete guide for Cursor AI to work effectively on MCP Korean Legislation
> **Philosophy**: Plan before code, test before complete, ask before breaking
> **Version**: 1.0 (2025-12-31)

---

## Table of Contents

1. [Core Principles](#1-core-principles)
2. [Decision Framework](#2-decision-framework)
3. [Development Environment Management](#3-development-environment-management)
4. [Project Reference](#4-project-reference)
5. [Task Checklists](#5-task-checklists)
6. [Code Patterns](#6-code-patterns)
7. [Prohibited Actions](#7-prohibited-actions)

---

## 1. Core Principles

### 1.0 Sensitive Data Protection (CRITICAL)

**Definition**: API keys, passwords, tokens, OC values

**NEVER**:
- Create `.env` backup files (`.env.bak`, `.env.backup`)
- Add sensitive files to git
- Expose API keys in logs or output
- Hardcode OC values or API keys in code

**Required Confirmation** (in Korean):
```
⚠️ "이 작업은 민감 정보(.env, API 키 등)를 포함합니다. 진행할까요?"
```

**Safe Alternatives**:
- Use `.env.example` with placeholders
- Reference via `os.environ.get('LEGISLATION_API_KEY')`
- Verify `.gitignore` includes sensitive files

### 1.1 Test Before Complete (CRITICAL)

**Completion Criteria**:
- Code alone is NOT complete
- MUST test before marking TODO as done

**Testing Procedure**:
1. API 호출: 실제 법제처 API 호출 테스트
2. Tool 동작: MCP tool이 정상적으로 등록되고 실행되는지 확인
3. Integration: 전체 플로우 테스트

**Correct Flow**:
```
1. Write/modify code
2. Restart service if needed
3. Execute actual test:
   - API: 실제 법제처 API 호출
   - Tool: MCP tool 실행 테스트
4. Verify result
5. ✅ No errors → Mark TODO complete
6. ❌ Errors found → Fix and retry from step 2
```

### 1.2 Plan Before Code (ALWAYS)

**Before Coding (15-20 min)**:
- Research best practices (web search)
- Review existing patterns in codebase
- Check API documentation in `docs/`
- Compare 2-3 approaches

**After Completion (5-10 min)**:
- Record learnings if significant
- Update documentation if needed

### 1.3 Required Reference Documents

**Priority 1: Project Documents**
- `AGENTS.md` — Project structure, workflows, architecture
- `docs/api-master-guide.md` — API 구조 및 패턴
- `docs/api-*.md` — 카테고리별 API 상세 가이드

**Priority 2: Skills Documents**
- `skills/tool-development.md` — Tool 개발 가이드
- `skills/api-integration.md` — API 통합 가이드
- `skills/cache-management.md` — 캐시 관리
- `skills/graph-search.md` — 지식 그래프 검색
- `skills/self-improvement.md` — 자가 개선 패턴

**Priority 3: Code Reference**
- `src/mcp_kr_legislation/utils/korean_law_api_complete_guide.md` — 전체 API 가이드 (참고용)

### 1.4 LLM 응답 최적화 (IMPORTANT)

**원칙**: MCP 도구 응답은 LLM에 피드되므로 토큰 효율성이 중요

**MUST**:
- 불필요한 이모지 사용 지양 (정보 전달에 필수적인 경우만 사용)
- 핵심 정보 위주로 포맷팅
- 중복 데이터 제거

**MUST NOT**:
- 응답이 길다고 임의로 내용을 줄이거나 생략하지 말 것
- 데이터 품질을 희생하여 응답 크기를 줄이지 말 것

**대용량 응답 처리 전략**:
1. **핵심 추출**: 전체 데이터에서 핵심 정보만 추출하여 요약본 제공
2. **캐시화**: 대용량 데이터는 캐시에 저장하고 요약본 반환
3. **분할 도구**: 필요시 목록/상세 조회를 별도 도구로 분리
4. **단계적 조회**: 요약 → 상세 순서로 조회할 수 있는 도구 구조

**Good Example**:
```
**법령명**: 개인정보보호법
**MST**: 248613
**시행일**: 2024-03-15
```

**Bad Example**:
```
🔍 **검색 결과** 🔍

📜 **법령명**: 개인정보보호법 ✨
📋 **MST**: 248613
📅 **시행일**: 2024-03-15 🎉

💡 더 많은 정보가 필요하시면 말씀해주세요! 😊
```

**응답 크기 가이드라인**:
- 목록 조회: 필수 필드 포함 (법령명, ID, 시행일 등)
- 상세 조회: 요약본 제공 후 전체 데이터는 캐시에서 조회 가능하도록
- 에러 메시지: 간결하게, 해결 방법 1-2개 제시

---

## 2. Decision Framework

### 2.1 When to Ask vs Proceed

**Ask User**:
- Task is ambiguous or has multiple valid interpretations
- Change affects existing working functionality
- Operation involves sensitive data
- Significant architectural decision
- Breaking change to public API

**Proceed Autonomously**:
- Task is clearly defined
- Following established patterns
- Adding new isolated feature
- Fixing obvious bugs
- Routine refactoring within single file

### 2.2 When to Use Existing Code vs Create New

**Pre-Code Checklist (MANDATORY)**:
Before writing ANY new function/component:
```
□ 1. Search codebase for similar functionality
     - grep for keywords related to the task
     - Check tools/, apis/, utils/
     
□ 2. Check if existing code can be:
     - Used directly (import and call)
     - Extended (add parameter, method)
     - Wrapped (thin wrapper for specific use)
     
□ 3. If creating new, ask yourself:
     - Why can't existing code be modified?
     - Will this create duplication?
     - Did I check ALL related files?
```

**Reuse Criteria**:
| Similarity | Action |
|------------|--------|
| 80%+ same | Use existing, modify if needed |
| 50-80% same | Extend existing or extract common |
| <50% same | Create new, document why |

**Anti-Patterns (PROHIBITED)**:
```
❌ Creating search_law2() when search_law() exists
❌ New formatting function when law_tools_utils.py exists  
❌ Duplicate API call logic in multiple files
❌ Copy-paste with minor modifications
```

**Correct Approach**:
```
✅ Import and use existing utilities
✅ Extend existing function with optional parameters
✅ Create shared component and import everywhere
✅ Extract common logic to utils layer
```

### 2.3 When to Refactor vs Patch

**Patch** (default for bug fixes):
- Minimal change to fix issue
- No surrounding code cleanup
- Keep PR small and focused

**Refactor** (only when requested):
- User explicitly asks for cleanup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChangooLee/mcp-kr-legislation](https://github.com/ChangooLee/mcp-kr-legislation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
