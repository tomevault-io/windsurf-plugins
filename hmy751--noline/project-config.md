---
trigger: always_on
description: > Noline: 네트워크가 없어도 여행은 계속된다. Selective Local-First 여행 관리 앱.
---

# Noline Project Guide

> Noline: 네트워크가 없어도 여행은 계속된다. Selective Local-First 여행 관리 앱.

이 파일은 루트 탐색 허브다. 상세 정책은 workspace guide, [rules](./.claude/rules/README.md), [guards](./.claude/guards/README.md), [runbooks](./.claude/runbooks/README.md), [context](./.claude/context/README.md)가 나누어 소유한다.

## Claude/Codex Bridge

- `CLAUDE.md`가 로컬 프로젝트 가이드의 원천이다.
- `AGENTS.md`는 `CLAUDE.md`를 가리키는 Codex bridge다.
- 같은 bridge가 적용되는 workspace:
  - [apps/client/CLAUDE.md](./apps/client/CLAUDE.md)
  - [apps/server/CLAUDE.md](./apps/server/CLAUDE.md)
  - [packages/schema/CLAUDE.md](./packages/schema/CLAUDE.md)
  - [packages/ui/CLAUDE.md](./packages/ui/CLAUDE.md)
- 하네스 구조와 tool bridge 규칙은 [Noline AI Harness](./.claude/harness/README.md)를 따른다.
- 실행층은 `.claude/skills/noline-work`, `.agents/skills/noline-work`, `.claude/agents/`, `.codex/agents/`가 얇게 맡는다. 기준 본문은 계속 rules/guards/runbooks/context가 소유한다.

## Start Here

| 필요 | 읽을 문서 |
| --- | --- |
| `.claude` 자료 역할 파악 | [Document Map](./.claude/README.md) |
| 하네스/bridge 작업 | [Noline AI Harness](./.claude/harness/README.md) |
| 작업 실행 dispatcher | [noline-work skill](./.claude/skills/noline-work/SKILL.md) |
| 코드 수정 중 지켜야 할 compact rule | [Noline Rules](./.claude/rules/README.md) |
| 코드 변경 전후 보호 정책 점검 | [Noline Guard Map](./.claude/guards/README.md) |
| 반복 작업 시작 순서 | [Noline Runbooks](./.claude/runbooks/README.md) |
| 깊은 아키텍처/기능 맥락 | [Noline Context Map](./.claude/context/README.md) |
| 하네스 레이어 개편 이유 | [Decision: Harness Layer Model](./.claude/decisions/2026-05-06-harness-layer-model.md) |
| 최근 하네스 실행 기록 | [Harness Execution Record](./.claude/audits/2026-05-06-harness-execution-plan.md) |

## Common Runbooks

| 작업 | 시작점 |
| --- | --- |
| 동기화 버그 수정 | [Debug Sync Issues](./.claude/runbooks/README.md#sync-debug) |
| 새 Entity 추가 | [Add New Entity](./.claude/runbooks/README.md#add-entity) |
| API 추가 | [Add API Endpoint](./.claude/runbooks/README.md#api-endpoint) |
| Form 구현 | [Implement Forms](./.claude/runbooks/README.md#form-pattern) |
| 날짜/시간 처리 | [Work With Dates/Times](./.claude/runbooks/README.md#datetime-utils) |
| 통화 표시 | [Display Currency/Amounts](./.claude/runbooks/README.md#currency-utils) |
| UI 컴포넌트 | [Build UI Components](./.claude/runbooks/README.md#component-guide) |

## Project Identity

Noline은 불안정한 네트워크 환경에서도 여행 계획과 경비 기록을 이어갈 수 있게 하는 모바일 앱이다.

현재 용어:

| 개념 | 현재 이름 | 과거/역사 문서의 이름 |
| --- | --- | --- |
| 아키텍처 | Selective Local-First | Local-First, Offline-First, Local-First Server-Aware |
| 라우터 | Activation Router | Offline-Prep Router, Hybrid Router |
| ID 전략 | Client-Side ID Generation | Echo Protocol |
| UX 비유 | 오프라인 보험 | 오프라인 구독 |

용어 변경의 이유는 [2026-03-21 terminology decision](./.claude/decisions/2026-03-21-terminology-unification.md)에 기록되어 있다. 역사 문서의 오래된 용어는 그대로 둘 수 있지만, 현재 정책 설명에서는 위 용어를 사용한다.

## Core Invariants

1. 활성화된 여행은 Local SQLite가 진실의 원천이다. 비활성 여행은 Server API가 진실의 원천이다.
2. Trip/Schedule/Expense 같은 Data Layer는 Activation Router를 통해 Local/Remote를 분기한다.
3. Map/Search/Directions 같은 Service Layer는 소유권과 sync 대상이 아니므로 Policy 기반 Network-First로 다룬다.
4. 로컬 mutation과 `sync_queue` 추가는 `withTransaction` 안에서 원자적으로 처리한다.
5. Entity 생성 ID는 클라이언트가 `generateId()`로 만들고, 서버는 클라이언트 ID를 수용한다.
6. `@repo/schema`가 클라이언트/서버 타입 계약의 원천이다. 타입은 schema에서 `z.infer`로 추론한다.
7. 시간 값은 ISO 8601 datetime with timezone으로 저장/전송하고, 표시는 shared util에서 처리한다.
8. delete flow는 soft delete와 cleanup 안전 조건을 따른다.
9. 서버 route는 인증과 user ownership을 확인한다.
10. 정책 제한 UI는 `useAppPolicy`, `PolicyErrorDisplay`, `NetworkStatusIndicator` 같은 기존 패턴을 먼저 사용한다.

상세 체크는 [Rules](./.claude/rules/README.md)와 [Guard Map](./.claude/guards/README.md)을 기준으로 한다.

## Task Rule Index

| 주제 | Rule |
| --- | --- |
| Local/Remote routing | [Activation Router](./.claude/rules/activation-router.md) |
| Local mutation + sync | [Transaction + Sync Queue](./.claude/rules/transaction-sync-queue.md) |
| Schema/type contract | [Schema First](./.claude/rules/schema-first.md) |
| Create ID strategy | [Client-Side ID](./.claude/rules/client-side-id.md) |
| 날짜/시간 | [ISO Time](./.claude/rules/iso-time.md) |
| 서버 인증/소유권 | [Auth/User Scope](./.claude/rules/auth-user-scope.md) |
| 정책 제한 UI | [Policy UI](./.claude/rules/policy-ui.md) |
| 깊은 맥락 | [Context Map](./.claude/context/README.md) |

## Workspace Ownership

| Workspace | 책임 | Guide |
| --- | --- | --- |
| `apps/client` | React Native, FSD, local DB, Router, policy UI, offline features | [apps/client/CLAUDE.md](./apps/client/CLAUDE.md) |
| `apps/server` | Express routes, auth/user scope, PostgreSQL, sync endpoints | [apps/server/CLAUDE.md](./apps/server/CLAUDE.md) |
| `packages/schema` | Zod entity/request/response contract | [packages/schema/CLAUDE.md](./packages/schema/CLAUDE.md) |
| `packages/ui` | 공유 UI primitives와 컴포넌트 철학 | [packages/ui/CLAUDE.md](./packages/ui/CLAUDE.md) |

## Project Structure

```text
noline/
├── apps/
│   ├── client/          # React Native + Expo app
│   └── server/          # Express API server
├── packages/
│   ├── schema/          # Shared Zod contracts
│   ├── ui/              # Shared UI components
│   ├── eslint-config/   # Shared lint config
│   └── typescript-config/ # Shared TypeScript config
├── .claude/
│   ├── rules/           # Compact task/path rules
│   ├── guards/          # High-cost failure map
│   ├── runbooks/        # Repeated task entry points

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hmy751/noline](https://github.com/hmy751/noline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
