---
trigger: always_on
description: OpenDocuments는 자체 호스팅 RAG(Retrieval-Augmented Generation) 플랫폼이다. GitHub, Notion, Google Drive 등 다양한 소스의 문서를 인덱싱하고, 자연어로 질문하면 출처와 함께 답변을 제공한다.
---

# AGENTS.md

## Project Overview

OpenDocuments는 자체 호스팅 RAG(Retrieval-Augmented Generation) 플랫폼이다. GitHub, Notion, Google Drive 등 다양한 소스의 문서를 인덱싱하고, 자연어로 질문하면 출처와 함께 답변을 제공한다.

- **Repository**: https://github.com/joungminsung/OpenDocuments
- **License**: MIT
- **Node.js**: >= 20.0.0
- **Package Manager**: npm@10.0.0
- **Build Orchestrator**: Turborepo v2.3.0
- **Language**: TypeScript 5.5+ (strict mode, ESM)

---

## Monorepo Structure

```
packages/          # 핵심 패키지 (5개)
  core/            # 비즈니스 로직 전체 (RAG, ingest, storage, auth, plugin system)
  server/          # HTTP API (Hono), MCP server, auth middleware, widget
  cli/             # CLI 17개 명령어 (Commander.js) - npm 글로벌 패키지 'opendocuments'
  web/             # React SPA (Vite + Tailwind) - private, npm 배포 안 함
  client/          # TypeScript SDK (@opendocuments/client)

plugins/           # 플러그인 (21개)
  model-*/         # 5개: ollama, openai, anthropic, google, grok
  parser-*/        # 9개: pdf, docx, xlsx, html, jupyter, email, code, pptx
  connector-*/     # 8개: github, notion, gdrive, s3, confluence, swagger, web-crawler, web-search

docs-site/         # VitePress 문서 사이트 (GitHub Pages 자동 배포)
```

---

## Common Commands

```bash
npm run setup          # install + build (처음 클론 시)
npm run build          # turbo build (전체)
npm run test           # turbo test (51 tasks, ~648 tests)
npm run dev            # watch mode (전체)
npm run typecheck      # turbo typecheck

# 특정 패키지만
npx turbo build --filter=opendocuments-core
npx turbo test --filter=opendocuments-core
```

---

## Code Conventions

### TypeScript

- **strict mode** 필수 (`tsconfig.base.json`)
- **ESM**: 모든 import에 `.js` 확장자 필수
  ```typescript
  import { sha256 } from './utils/hash.js'    // O
  import { sha256 } from './utils/hash'        // X
  ```
- `any` 사용 금지 (web UI 제외). `unknown` 또는 명시적 타입 사용
- 공개 API에는 JSDoc 작성

### Naming

| 대상 | 규칙 | 예시 |
|------|------|------|
| 파일 | kebab-case | `document-store.ts` |
| 클래스 | PascalCase | `DocumentStore` |
| 함수/변수 | camelCase | `getDocumentBySourcePath` |
| 상수 | UPPER_SNAKE_CASE | `MAX_ALERTS` |
| DB 컬럼 | snake_case | `workspace_id` |
| 환경변수 | UPPER_SNAKE_CASE | `OPENDOCUMENTS_DATA_DIR` |
| CLI 명령어 | kebab-case | `auth create-key` |

### CLI Output

- **이모지 금지** - `log` 유틸리티의 ANSI 기호 사용:
  ```typescript
  import { log } from '@opendocuments/core'
  log.ok('Success')      // [ok]  green
  log.fail('Error')      // [!!]  red
  log.info('Info')        // [--]  blue
  log.arrow('Next step')  // [->]  cyan
  log.wait('Processing') // [..]  yellow
  ```

### Error Handling

- 중요 경로에서 에러를 삼키지 말 것 - 최소한 로깅
- 사용자 대상 에러는 actionable하게 (무엇이 잘못됐고 + 어떻게 고치는지)
- 프로덕션에서 내부 세부사항 노출 금지 (stack trace, 내부 경로 등)

### Security

- API 키/시크릿 하드코딩 금지 - 환경변수 사용
- SQL 쿼리는 parameterized statements (`?`)
- LanceDB 필터는 `buildWhereClause()` 사용 (raw string interpolation 금지)
- FTS5 쿼리는 `escapeFTS5Query()` 사용 (raw user input 금지)
- team mode에서 새 HTTP 엔드포인트는 auth middleware 보호 필수
- 에러 응답에 stack trace 포함 금지

---

## Testing

- **Framework**: Vitest (Jest-compatible)
- **파일 위치**: 각 패키지 내 `tests/` 디렉토리
- **파일명**: `*.test.ts`
- **globals 활성화**: `describe`, `it`, `expect`, `vi` import 불필요
- **DB 테스트**: 실제 SQLite (`:memory:`) + 실제 LanceDB (temp dir) 사용
- **외부 API**: `vi.stubGlobal('fetch', ...)` 로 mock
- 테스트 간 독립성 유지 (shared state 금지, `beforeEach`에서 초기화)
- happy path + error path 모두 테스트
- 리소스 정리 필수 (`afterEach`에서 DB close, temp dir 삭제)

---

## Commit Messages

[Conventional Commits](https://www.conventionalcommits.org/) 사용:

```
<type>(<scope>): <description>
```

**Types**: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`, `ci`

**Scopes**: `core`, `server`, `cli`, `web`, `client`, 또는 생략 (다중 패키지)

```
feat(core): add intent classification
fix(server): escape FTS5 query input
docs: update README with use cases
```

---

## Versioning & Release

[Changesets](https://github.com/changesets/changesets) 사용.

### Package Names & npm 배포

| 패키지 | npm 이름 | 현재 버전 |
|--------|----------|-----------|
| core | `opendocuments-core` | 0.2.0 |
| server | `opendocuments-server` | 0.2.0 |
| cli | `opendocuments` (글로벌 bin) | 0.2.1 |
| client | `@opendocuments/client` | 0.2.0 |
| web | `@opendocuments/web` | private (배포 안 함) |
| model-* | `opendocuments-model-{provider}` | 0.1.0~0.1.2 |
| parser-* | `opendocuments-parser-{format}` | 0.1.0 |
| connector-* | `@opendocuments/connector-{type}` | 0.1.0 |

### Version Bump 기준

| 범위 | 조건 | 예시 |
|------|------|------|
| patch (0.1.**x**) | 버그 수정, 문서 업데이트 | FTS5 쿼리 이스케이핑 수정 |
| minor (0.**x**.0) | 새 기능, 새 플러그인 | Google Drive connector 추가 |
| major (**x**.0.0) | 브레이킹 변경 | 플러그인 인터페이스 변경 |

### Plugin Compatibility

모든 플러그인은 `coreVersion: '^0.1.0'` 선언. core minor 업데이트 시 기존 플러그인과 하위 호환성 유지 필수.

---

## Version Release Checklist

버전을 올릴 때 아래 단계를 순서대로 수행한다.

### Phase 1: 코드 준비

- [ ] feature 브랜치의 모든 작업이 완료되었는지 확인
- [ ] `npm run build` 성공 확인
- [ ] `npm run typecheck` 성공 확인
- [ ] `npm run test` 전체 통과 확인 (51 tasks, ~648 tests)

### Phase 2: 코드 리뷰 (3회 반복)

최종 품질 보장을 위해 전체 변경 사항에 대해 코드 리뷰를 3회 반복한다.

**1차 리뷰 - 기능 및 정확성:**
- [ ] 변경된 모든 파일의 로직이 정확한지 확인
- [ ] 엣지 케이스 처리 확인
- [ ] 에러 핸들링이 적절한지 확인
- [ ] 새 기능에 테스트가 있는지 확인
- [ ] 발견된 이슈 수정

**2차 리뷰 - 보안 및 성능:**
- [ ] SQL injection, XSS 등 보안 취약점 점검

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joungminsung/OpenDocuments](https://github.com/joungminsung/OpenDocuments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
