---
trigger: always_on
description: TypeScript-first ReBAC(Relation-Based Access Control) 툴킷. Yarn workspaces 모노레포(`yarn@4.12.0`).
---

# Graplix – Claude Code Guide

## 프로젝트 개요

TypeScript-first ReBAC(Relation-Based Access Control) 툴킷. Yarn workspaces 모노레포(`yarn@4.12.0`).

```
packages/
  language/         # Langium 기반 .graplix 스키마 파서·검증기
  engine/           # 런타임 권한 평가 엔진
  codegen/          # .graplix → TypeScript 코드 생성 CLI
  vscode-extension/ # VS Code 언어 지원
```

패키지 매니저: **yarn@4.12.0** (pnpm·npm 사용 금지).

### 주요 파일 맵

- 루트: `package.json`, `tsconfig.json`, `biome.json`
- Language: `packages/language/src/graplix.langium`, `src/validator.ts`, `src/parse.ts`
- Engine: `packages/engine/src/buildEngine.ts`, `src/private/*`, `src/buildEngine.spec.ts`
- Codegen: `packages/codegen/src/generate.ts`, `src/cli.ts`, `src/config.ts`
- Extension: `packages/vscode-extension/src/extension.ts`, `src/language-server.ts`

---

## 빌드·테스트·포맷 명령

### 루트

```bash
yarn build    # 전체 워크스페이스 빌드 (ultra -r build)
yarn format   # Biome 포맷/린트 자동수정 (biome check --fix --unsafe)
yarn test     # 전체 테스트
```

### Language (`@graplix/language`)

```bash
yarn workspace @graplix/language langium:generate          # 문법 출력 재생성
yarn workspace @graplix/language build                     # langium:generate 후 tsdown
yarn workspace @graplix/language test
```

### Engine (`@graplix/engine`)

```bash
yarn workspace @graplix/engine build
yarn workspace @graplix/engine test
yarn workspace @graplix/engine vitest run src/buildEngine.spec.ts
yarn workspace @graplix/engine vitest run src/buildEngine.spec.ts -t "explain"
```

### Codegen (`@graplix/codegen`)

```bash
yarn workspace @graplix/codegen build
yarn workspace @graplix/codegen test
yarn workspace @graplix/codegen codegen ./schema.graplix
yarn workspace @graplix/codegen vitest run src/generate.spec.ts
```

### VS Code Extension (`graplix-vscode-extension`)

```bash
yarn workspace graplix-vscode-extension build   # 번들 빌드 및 VSIX 패키징
yarn workspace graplix-vscode-extension watch
# test 스크립트 없음
```

### 검증 순서 (권장)

1. 변경된 패키지의 테스트 실행
2. 변경된 패키지의 빌드 실행
3. 문법/언어 서비스 변경 시: `@graplix/language build` → `graplix-vscode-extension build`
4. 최종: `yarn build`

---

## Engine 패키지 아키텍처 (`packages/engine`)

### 핵심 타입

| 타입 | 위치 | 설명 |
|------|------|------|
| `EntityRef` | `src/EntityRef.ts` (공개 export) | `class EntityRef { type, id }` — 내부 엔티티 표현. Symbol 브랜드로 고유성 보장 |
| `Query<TContext, TEntityInput>` | `src/Query.ts` | `check`/`explain` 입력. `user`, `object`는 `TEntityInput`. `context`는 필수 |
| `CheckEdge` | `src/CheckEdge.ts` | `explain` 출력의 엣지. `from`, `to`가 `EntityRef` 인스턴스 |
| `GraplixEngine<TContext, TEntityInput>` | `src/GraplixEngine.ts` | `check(query)`, `explain(query)` 메서드 |
| `BuildEngineOptions<TContext>` | `src/BuildEngineOptions.ts` | `buildEngine` 생성 옵션 |
| `ResolverInfo` | `src/ResolverInfo.ts` | resolver 호출 시 전달되는 메타. `signal: AbortSignal` 포함 |

### EntityRef 규칙

`EntityRef`는 내부 전용 클래스입니다. **공개 API(`check`/`explain`)에 EntityRef를 직접 넘기지 않습니다.** 사용자는 항상 도메인 엔티티를 `TEntityInput`으로 전달합니다.

`CheckEdge.from`/`to`는 `EntityRef` 타입이므로, explain 결과를 타입 명시적으로 다루려면 import 가능합니다.

```typescript
import type { EntityRef } from "@graplix/engine";
```

### 도메인 엔티티 전달 방식

```typescript
// ✅ 도메인 엔티티를 TEntityInput으로 직접 전달
const engine = await buildEngine<MyContext, User | Repository>({ ... });
await engine.check({
  user: userEntity,       // User 타입
  object: repoEntity,     // Repository 타입
  relation: "owner",
  context: myContext,     // TContext (필수)
});

// ❌ EntityRef 직접 전달 불가 (Query.user/object는 TEntityInput만 허용)
// ❌ "type:id" 문자열 방식 (제거됨)
```

### Resolver 인터페이스

```typescript
interface Resolver<TEntity, TContext> {
  id(entity: TEntity): string;
  load(id: string, context: TContext, info: ResolverInfo): Promise<TEntity | null>;
  relations?: {
    [relation: string]: (
      entity: TEntity,
      context: TContext,
      info: ResolverInfo,
    ) => TEntity | TEntity[] | null | Promise<...>;
  };
}
```

**relation resolver는 도메인 엔티티를 반환합니다.** EntityRef 인스턴스를 반환하지 않습니다. 엔진이 `resolveType` 또는 schema 타입 힌트(`allowedTargetTypes`)를 통해 타입을 자동 판별합니다.

### resolveType

```typescript
type ResolveType<TContext> = (value: unknown, context: TContext) => string | null;
```

- **동기 함수**이며 **필수**입니다.
- 전달된 값의 Graplix 타입명을 반환합니다. 판별 불가 시 `null` 반환.
- `null`을 반환하면 schema 타입 힌트(allowedTargetTypes) 경로로 폴백합니다.
- relation resolver 출력에는 schema에서 타입이 이미 알려져 있으므로 `resolveType`이 `null`을 반환해도 동작합니다.
- **`query.user`/`query.object`에 대해서는 반드시 올바른 타입명을 반환해야 합니다.**

```typescript
// ✅ 구조적 필드로 타입 판별
const resolveType: ResolveType<MyContext> = (value) => {
  if (typeof value !== "object" || value === null) return null;
  const v = value as Record<string, unknown>;
  if ("reporterId" in v) return "issue";
  if ("adminIds" in v) return "organization";
  return "user";
};
```

### private/ 디렉터리 구조

```
src/
  EntityRef.ts            # class EntityRef — 내부 엔티티 표현 (public export)

src/private/
  toEntityRef.ts          # 도메인 엔티티 → EntityRef 변환 (resolveType → schema 힌트)
  toEntityRefList.ts      # 복수 값 변환 (동기)
  evaluateRelation.ts     # 관계 평가 진입점 + 사이클 감지
  evaluateRelationTerm.ts # 개별 term(direct/from) 평가
  getRelationValues.ts    # resolver 호출, 타임아웃, 캐싱
  loadEntity.ts           # 엔티티 로드 및 캐싱
  entityMatches.ts        # EntityRef 동등 비교
  getStateKey.ts          # URLSearchParams 기반 캐시 키 생성
  InternalState.ts        # 평가 중 공유 상태

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daangn/graplix](https://github.com/daangn/graplix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
