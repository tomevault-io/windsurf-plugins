---
trigger: always_on
description: 백엔드 테스트 작성 규칙 (Vitest, Domain Entity, UseCase 테스트 패턴)
---

# 🧪 백엔드 테스트 작성 규칙

## 테스트 프레임워크
- **Vitest**: 모든 백엔드 테스트에 Vitest 사용
- **BDD 스타일**: describe/it 구조로 시나리오 중심 테스트 작성
- **globals: true**: vitest.config.ts에서 전역 함수 활성화

## 테스트 파일 구조
```
libs/{domain}/{layer}/src/
├── libs/
│   ├── entities/
│   │   └── User.ts
│   ├── usecases/
│   │   └── CreateUser.ts
│   └── repositories/
│       └── UserRepository.ts
└── tests/
    ├── fixtures/                    # 재사용 가능한 테스트 헬퍼
    │   ├── createMockUserRepository.ts
    │   ├── createTestUser.ts
    │   └── ...
    ├── entities/
    │   └── User.spec.ts
    └── usecases/
        └── CreateUser.spec.ts
```

## Domain Entity 테스트 패턴

### 테스트 범위
- `create()` 팩토리 메서드의 기본값 검증
- `reconstitute()` 데이터 복원 검증
- 도메인 메서드의 상태 변경 검증
- **제외**: TypeScript로 보장되는 readonly 속성 검증 (불필요)

### Entity 테스트 예시
```typescript
import { describe, expect, it } from 'vitest';
import { WikiDocument } from '../../libs/entities/WikiDocument';

describe('WikiDocument', () => {
  describe('create', () => {
    it('새 문서 생성 시 기본값이 올바르게 설정된다', () => {
      const document = WikiDocument.create({
        workspaceId: 'wks_123',
        title: 'Test',
        content: 'Content',
        createdById: 'usr_456',
      });

      expect(document.id).toMatch(/^wdc_/); // ID prefix 검증
      expect(document.version).toBe(1);     // 기본값 검증
      expect(document.viewCount).toBe(0);
      expect(document.deletedAt).toBeNull();
    });
  });

  describe('reconstitute', () => {
    it('기존 문서 데이터로부터 엔티티를 복원한다', () => {
      const props = {
        id: 'wdc_123',
        // ... 모든 속성
      };
      const document = WikiDocument.reconstitute(props);

      expect(document.id).toBe(props.id);
      // 각 속성이 정확히 복원되었는지 검증
    });
  });

  describe('incrementVersion', () => {
    it('버전이 1 증가하고 수정자가 업데이트된다', () => {
      const document = WikiDocument.create({...});

      document.incrementVersion('usr_789');

      expect(document.version).toBe(2);
      expect(document.lastModifiedById).toBe('usr_789');
    });
  });
});
```

## Service UseCase 테스트 패턴

### Fixtures 작성 규칙

#### Mock Repository
```typescript
// createMockUserRepository.ts
import { UserRepository } from '@heewon.dev/user-domain';
import { vi } from 'vitest';

export function createMockUserRepository(): UserRepository {
  return {
    findById: vi.fn(),
    findByExternalId: vi.fn(),
    update: vi.fn(),
    save: vi.fn(),
  };
}

// Repository update 패턴용 헬퍼
export function mockUserRepositoryUpdate(repo: UserRepository, user: User) {
  vi.mocked(repo.update).mockImplementation(async (_id, modifier) => {
    return modifier(user);
  });
}
```

#### Test Factory
```typescript
// createTestUser.ts
import { User } from '@heewon.dev/user-domain';

export function createTestUser(
  overrides?: Partial<Parameters<typeof User.create>[0]>
) {
  return User.create({
    externalId: 'clerk_test_user_123',
    ...overrides,
  });
}
```

### UseCase 테스트 예시
```typescript
import { describe, expect, it, vi } from 'vitest';
import { CreateWorkspace } from '../../libs/usecases/CreateWorkspace';
import {
  createMockUserRepository,
  mockUserRepositoryUpdate
} from '../fixtures/createMockUserRepository';
import { createMockWorkspaceRepository } from '../fixtures/createMockWorkspaceRepository';
import { createTestUser } from '../fixtures/createTestUser';

describe('CreateWorkspace', () => {
  describe('execute', () => {
    it('워크스페이스를 생성하고 사용자에게 할당한다', async () => {
      // Given: Mock 및 테스트 데이터 준비
      const workspaceRepo = createMockWorkspaceRepository();
      const userRepo = createMockUserRepository();
      const testUser = createTestUser();
      mockUserRepositoryUpdate(userRepo, testUser);

      const usecase = new CreateWorkspace(workspaceRepo, userRepo);

      // When: UseCase 실행
      const result = await usecase.execute({
        userId: 'usr_123',
        workspaceName: 'My Workspace',
      });

      // Then: 결과 및 호출 검증
      expect(workspaceRepo.save).toHaveBeenCalledOnce();
      expect(userRepo.update).toHaveBeenCalledOnce();
      expect(result.name).toBe('My Workspace');
      expect(testUser.workspaceId).toBe(result.id);
    });
  });
});
```

## 테스트 작성 원칙

### 1. Given-When-Then 구조
```typescript
it('설명', async () => {
  // Given: 테스트 준비 (Mock, 데이터 생성)
  const repo = createMockRepository();
  const testData = createTestData();

  // When: 실행
  const result = await usecase.execute(input);

  // Then: 검증
  expect(result).toBe(expected);
});
```

### 2. 테스트 이름 규칙
- **한글 사용**: 명확한 시나리오 설명
- **행위 중심**: "~한다", "~를 반환한다"
- **구체적**: "문서가 업데이트되고 새 버전이 생성된다"

### 3. Mock은 최소한으로
- **필요한 메서드만**: 전체 객체보다는 필요한 인터페이스만
- **행위 검증**: `toHaveBeenCalledOnce()`, `toHaveBeenCalledWith()`
- **헬퍼 활용**: 반복되는 mock 패턴은 fixture로 추출

### 4. 테스트 독립성
- **각 테스트는 독립적**: 다른 테스트에 영향 없이 실행 가능
- **공유 상태 금지**: 테스트 간 변수 공유 금지
- **Setup/Teardown**: beforeEach/afterEach 최소화

## ID 검증 패턴

### IdPrefixes.validate() 사용
```typescript
import { IdPrefixes } from '@heewon.dev/provider-gid';

// Entity 테스트에서 ID 검증
it('올바른 ID prefix를 가진다', () => {
  const user = User.create({ externalId: 'clerk_123' });
  expect(IdPrefixes.USER.validate(user.id)).toBe(true);
});

// UseCase에서 ID 검증
if (!IdPrefixes.USER.validate(userId)) {
  throw new Error('Invalid user ID');
}
```

## 테스트하지 않아야 할 것

### ❌ TypeScript로 보장되는 것
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kang-heewon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
