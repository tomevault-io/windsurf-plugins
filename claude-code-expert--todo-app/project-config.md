---
trigger: always_on
description: > **핵심 원칙은 `.specify/memory/constitution.md` 참조**
---

# CLAUDE.md - Tika Development Guide

> **핵심 원칙은 `.specify/memory/constitution.md` 참조**
> 이 문서는 구체적인 구현 방법과 실무 가이드를 다룬다.

## 프로젝트 개요
Tika는 티켓 기반 칸반 보드 TODO 앱이다.
Next.js App Router 기반으로, 프런트엔드와 백엔드를 디렉토리 수준에서 분리한다.
src/shared/에서 타입과 검증 스키마를 공유한다.

## 프로젝트 구조
```
tika/
├── .claude/
│   ├── skills/       # Custom skills (디렉토리 + SKILL.md 형식)
│   │   └── changelog/SKILL.md
│   ├── commands/     # Legacy 커맨드 (단일 .md 파일, 여전히 작동)
│   │   └── speckit.*.md
│   └── settings.local.json
├── app/api/          # 백엔드 진입점 (Route Handlers)
├── src/
│   ├── server/       # 백엔드 로직 (services, db, middleware)
│   ├── client/       # 프런트엔드 로직 (components, hooks, api)
│   └── shared/       # 공유 타입, Zod 스키마, 상수
└── docs/             # 프로젝트 명세 문서
```

### .claude/ 디렉토리 구조 (Claude Code 전용)
- **`.claude/skills/`**: 권장 형식, 디렉토리 + `SKILL.md` + 지원 파일
  - 예: `.claude/skills/changelog/SKILL.md`
  - YAML frontmatter 필수 (name, description, user-invocable 등)
- **`.claude/commands/`**: 레거시 형식, 단일 `.md` 파일 (여전히 작동)
  - 예: `.claude/commands/speckit.plan.md`
  - Skills보다 기능이 제한적이지만 간단한 용도로 사용 가능
- **공식 문서**: https://code.claude.com/docs/skills.md

## 기술 스택
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript (strict mode)
- **Frontend**: React 19
- **Styling**: Tailwind CSS 4
- **Drag & Drop**: @dnd-kit/core + @dnd-kit/sortable
- **ORM**: Drizzle ORM 0.38.x
- **DB**: PostgreSQL (로컬: node-postgres, 배포: Vercel Postgres)
- **Validation**: Zod
- **Testing**: Jest + React Testing Library
- **Deployment**: Vercel

## MCP Servers (Model Context Protocol)

### Context7 - 공식 문서 자동 참조 🎯

**목적**: 추측 금지, 공식 문서 우선 원칙을 자동화

**기능**:
- 최신 라이브러리 문서를 실시간으로 fetch (Drizzle 0.38.x, React 19, Next.js 15 등)
- 할루시네이션 방지 (훈련 데이터가 아닌 공식 소스에서 직접)
- 버전별 정확한 API 참조
- 코드 예제, 스키마 정보, 마이그레이션 가이드 제공

**설정**:
```bash
# 1. API 키 발급: https://context7.com
# 2. .env.local에 추가
CONTEXT7_API_KEY=your-api-key-here

# 3. .mcp.json 확인 (이미 설정됨)
```

**사용법**:
```bash
# 명시적 사용
> use context7 to show me Drizzle ORM 0.38.x migration syntax

# 특정 라이브러리
> use context7 with @vercel/next to explain App Router caching

# 일반 질문 (자동 참조)
> How to validate with Zod in TypeScript strict mode?
```

**Documentation First 원칙 적용**:
- ✅ 구현 전: "use context7"로 최신 공식 문서 확인
- ✅ 불확실 시: Context7이 자동으로 올바른 방법 제시
- ✅ 검증: 공식 소스에서 가져온 정보이므로 신뢰 가능

**비용**: 무료 1,000 요청/월 (로컬 서버 사용 시 무제한)

**공식 문서**: https://context7.com/docs/clients/claude-code

## 환경 설정

### 환경 변수
```bash
# .env.local
DATABASE_URL=postgresql://user:password@localhost:5432/tika
```

### 경로 별칭
- `@/` → `src/`
- `@/app/` → `app/`
- `@/shared/` → `src/shared/`
- `@/server/` → `src/server/`
- `@/client/` → `src/client/`

## 명세 문서 (구현 전 필수 확인)
| 문서 | 용도 |
|------|------|
| docs/PRD.md | 제품 요구사항 |
| docs/TRD.md | 기술 요구사항 |
| docs/REQUIREMENTS.md | 상세 요구사항 (FR + NFR + US) |
| docs/API_SPEC.md | API 엔드포인트 명세 |
| docs/DATA_MODEL.md | DB 스키마, ERD, 비즈니스 규칙 |
| docs/COMPONENT_SPEC.md | 컴포넌트 계층, Props, 이벤트 |
| docs/TEST_CASES.md | TDD용 테스트 케이스 정의 |

## 코딩 컨벤션

### TypeScript
```typescript
// ✅ Good
interface Ticket {
  id: number;
  title: string;
}

export const TICKET_STATUS = {
  BACKLOG: 'BACKLOG',
  TODO: 'TODO',
} as const;

type TicketStatus = typeof TICKET_STATUS[keyof typeof TICKET_STATUS];

// ❌ Bad
interface ITicket { ... }           // I 접두사 사용 금지
enum TicketStatus { ... }           // enum 대신 const 객체 사용
let data: any;                      // any 사용 금지
```

### 백엔드 (app/api/ + src/server/)

#### Route Handler 패턴
```typescript
// app/api/tickets/route.ts
import { createTicketSchema } from '@/shared/validations/ticket';
import { ticketService } from '@/server/services/ticketService';

export async function POST(request: Request) {
  // 1. 요청 파싱
  const body = await request.json();

  // 2. Zod 검증
  const result = createTicketSchema.safeParse(body);
  if (!result.success) {
    return Response.json(
      { error: { code: 'VALIDATION_ERROR', message: result.error.message } },
      { status: 400 }
    );
  }

  // 3. 서비스 호출
  const ticket = await ticketService.create(result.data);

  // 4. 응답 반환
  return Response.json(ticket, { status: 201 });
}
```

#### 서비스 레이어 패턴
```typescript
// src/server/services/ticketService.ts
import { db } from '@/server/db';
import { tickets } from '@/server/db/schema';
import type { CreateTicketInput, Ticket } from '@/shared/types';

export const ticketService = {
  async create(input: CreateTicketInput): Promise<Ticket> {
    // 비즈니스 로직
    const position = await this.calculatePosition(input.status);

    // DB 쿼리
    const [ticket] = await db
      .insert(tickets)
      .values({ ...input, position })
      .returning();

    return ticket;
  },

  async calculatePosition(status: string): Promise<number> {
    // 복잡한 로직은 별도 메서드로 분리
    const lastTicket = await db
      .select()
      .from(tickets)
      .where(eq(tickets.status, status))
      .orderBy(desc(tickets.position))
      .limit(1);

    return lastTicket[0]?.position ?? 0 - 1024;
  },
};
```

#### 에러 응답 형식
```typescript
// ✅ 올바른 에러 응답
return Response.json(
  {
    error: {
      code: 'TICKET_NOT_FOUND',
      message: '티켓을 찾을 수 없습니다'
    }
  },
  { status: 404 }
);

// ❌ 잘못된 에러 응답
return Response.json({ message: 'Not found' }, { status: 404 });
return Response.json({ error: 'Not found' }, { status: 404 });
```

### 프런트엔드 (src/client/)

#### 컴포넌트 패턴

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [claude-code-expert/todo-app](https://github.com/claude-code-expert/todo-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
