---
trigger: always_on
description: > 이 프로젝트는 Cloudflare Workers 기반 SaaS 보일러플레이트다. 복사해서 새 프로젝트를 시작하는 용도로 설계되었다. 이 문서는 Codex, Claude Code 등 에이전트가 이 보일러플레이트를 올바르게 이해하고 커스터마이징을 도울 수 있도록 작성되었다.
---

# 옥토워커스 — 에이전트 작업 가이드

> 이 프로젝트는 Cloudflare Workers 기반 SaaS 보일러플레이트다. 복사해서 새 프로젝트를 시작하는 용도로 설계되었다. 이 문서는 Codex, Claude Code 등 에이전트가 이 보일러플레이트를 올바르게 이해하고 커스터마이징을 도울 수 있도록 작성되었다.

## 프로젝트 구조

- `/` — 루트 monorepo (pnpm workspace)
- `/worker/` — Hono + Cloudflare Workers API 서버
- `/apps/landing/` — 공개 랜딩 페이지 (React + Vite)
- `/apps/admin/` — 인증 기반 운영 콘솔 (React + Vite)
- `/packages/com/` — 프론트-백엔드 공유 타입 패키지
- `/_templates/` — 새 모듈 생성용 템플릿
- `/docs/` — 운영 문서

## 보일러플레이트 커스터마이징 가이드

> 이 프로젝트를 복사한 후 새 SaaS를 시작할 때 아래 순서로 진행한다.

### 1단계: 프로젝트 이름 교체

- `package.json` (루트): `octoworkers` → 새 프로젝트명
- `worker/package.json`: `@octoworkers/worker` → `@새프로젝트/worker`
- `apps/landing/package.json`, `apps/admin/package.json`: 패키지명 교체
- `packages/com/package.json`: `@octoworkers/com` → `@새프로젝트/com`
- `pnpm-workspace.yaml`은 구조 변경이 없으면 그대로 유지

### 2단계: Cloudflare 리소스 설정

`worker/wrangler.jsonc`에서 아래 값을 실제 값으로 교체:

```
REPLACE_WITH_ACCOUNT_ID        → Cloudflare Account ID
REPLACE_WITH_D1_DATABASE_ID    → D1 데이터베이스 ID
REPLACE_WITH_IMAGES_DELIVERY_HASH → Cloudflare Images delivery hash
example.com / admin.example.com → 실제 도메인
```

staging 환경도 동일하게 교체. `routes` 섹션은 DNS 설정 완료 후 주석 해제.

### 3단계: 시크릿 설정

```bash
cp worker/.dev.vars.example worker/.dev.vars
# .dev.vars에 실제 시크릿 값 입력
```

배포용 시크릿:

```bash
wrangler secret put ADMIN_LOGIN_PASSWORD
wrangler secret put ADMIN_JWT_SECRET
wrangler secret put AI_PROVIDER_API_KEY          # 선택
wrangler secret put CLOUDFLARE_IMAGES_API_TOKEN  # 선택
```

### 4단계: DB 초기화

```bash
pnpm --filter @octoworkers/worker db:migrate:local
```

기본 스키마: `site_settings`(싱글톤), `leads`, `media_assets`. 새 테이블은 `worker/migrations/0002_*.sql`로 추가.

### 5단계: 불필요한 예제 제거 (선택)

보일러플레이트에 포함된 예제 모듈 중 불필요한 것을 제거할 수 있다:

| 모듈 | 경로 | 제거 시 확인 |
| --- | --- | --- |
| KV/R2/WS 예제 | `worker/src/biz/ext/`, `apps/admin/src/biz/ext/` | `index.ts`에서 `extRoutes` 제거 |
| 에이전트 | `worker/src/biz/agt/` | `index.ts`에서 `agentRoutes` + `OpsAgent` export 제거, `wrangler.jsonc` DO 설정 제거 |
| 벡터 검색 | `worker/src/biz/vec/` | `index.ts`에서 `vectorRoutes` 제거, `wrangler.jsonc` vectorize 설정 제거 |
| AI 카피 | `worker/src/biz/aid/` | `index.ts`에서 `aiRoutes` 제거 |

### 6단계: 비즈니스 모듈 추가

`_templates/README.md` 참고. 요약:

1. `_templates/biz-module/` 복사 → `worker/src/biz/{3글자}/`
2. `__ITEM__` 플레이스홀더 치환
3. `packages/com/src/contracts.ts`에 공유 타입 추가
4. `worker/src/index.ts`에 `.route()` 등록
5. D1 마이그레이션 SQL 작성
6. `worker/test/app.test.ts`에 테스트 추가
7. `apps/admin/src/biz/{3글자}/` 에 훅/컴포넌트 추가

## 개발 규칙

> 경고: 이 규칙을 읽지 않고 개발을 진행하면 안 된다.

- `AGENTS.md`와 `CLAUDE.md`는 항상 동일한 기준 문서로 유지한다. 한쪽이 바뀌면 다른 한쪽도 반드시 즉시 양방향 동기화한다.
- 이 문서는 Codex와 Claude를 포함한 에이전트가 이 저장소에서 안전하게 작업하기 위한 공용 운영 문서다.
- 기능 추가, 구조 변경, 정책 변경, API 변경이 있으면 관련 문서를 즉시 갱신한다.
- 공통으로 재사용되는 로직은 중복 복사하지 말고 공통 모듈(`com/`)로 정리한다.
- 모듈, 서비스는 책임이 명확해야 하며 결합도는 낮추고 응집도는 높인다.
- 변경사항이 생기면 해당 파일만 국소적으로 보지 말고 전체 흐름에서 점검한다.
- 수정 후에는 입력, 처리, 저장, 출력까지 이어지는 사용자 흐름을 기준으로 영향을 다시 확인한다.
- 모든 개발 전후에는 반드시 영향분석을 수행한다.
- 변경이 작아 보여도 공통 모듈, 데이터 흐름, API, UI에 미치는 영향을 매번 확인한다.

## 기술 스택

| Layer | Technology |
| --- | --- |
| Language | TypeScript (strict) |
| API | Hono 4 on Cloudflare Workers |
| Frontend | React 19 + Vite (SWC), 두 개 앱 |
| Database | Cloudflare D1 (SQLite) |
| Storage | KV (캐시), R2 (오브젝트, 선택) |
| Media | Cloudflare Images (direct upload, 선택) |
| AI | Workers AI + AI Gateway + Vectorize (선택) |
| Realtime | Durable Objects + WebSocket (선택) |
| Testing | Vitest |
| Package Manager | pnpm 9.15 (workspace monorepo) |
| CI/CD | GitHub Actions → Cloudflare Workers |

## 실행 방법

```bash
pnpm install
cp worker/.dev.vars.example worker/.dev.vars
pnpm --filter @octoworkers/worker db:migrate:local
pnpm dev                    # 전체 (landing:5173 + admin:5174 + worker:8787)
```

```bash
pnpm dev:remote             # 원격 Cloudflare 리소스 연결 (AI/Vectorize/Agents)
pnpm dev:worker             # Worker만
pnpm dev:apps               # 프론트엔드만
pnpm check                  # tsc --noEmit (전 패키지)
pnpm test                   # Vitest
pnpm build                  # landing + admin 프로덕션 빌드
pnpm deploy:staging         # 빌드 → D1 마이그레이션 → staging 배포
pnpm deploy:prod            # 빌드 → D1 마이그레이션 → production 배포
```

## 아키텍처

```text
Browser
  → Cloudflare Workers (worker/src/index.ts)
    → 미들웨어: applySecurityHeaders → enforceAdminAccess → agentsMiddleware → cors → etag
    → 라우트:
        /api/health           → hlt/ (헬스체크)
        /api/auth/*           → aut/ (JWT 세션)
        /api/public/*         → pub/ (bootstrap, lead submit)
        /api/admin/dashboard  → dsh/ (통계)
        /api/admin/settings   → set/ (사이트 설정 CRUD)
        /api/admin/leads      → led/ (리드 CRM: 목록, 상세, 상태, 태그, 노트)
        /api/admin/images     → med/ (Cloudflare Images)
        /api/admin/ai         → aid/ (AI Gateway 카피 생성)
        /api/admin/email      → eml/ (이메일 템플릿, 발송, 로그)
        /api/admin/pages      → pag/ (콘텐츠 CMS: 페이지 CRUD, 발행)
        /api/admin/agt        → agt/ (Durable Object 에이전트)
        /api/admin/search     → srh/ (SQL LIKE 전문 검색)
        /api/admin/vec        → vec/ (Vectorize 시맨틱 검색)
        /api/admin/users      → usr/ (사용자 관리: CRUD, 역할, 활성화)
        /api/admin/logs       → log/ (접속 로그, API 로그, 시스템 통계)
        /api/admin/ext        → ext/ (KV/R2/WS 예제)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johunsang/octoworkers](https://github.com/johunsang/octoworkers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
